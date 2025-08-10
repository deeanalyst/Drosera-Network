## 1. Update Droserap CLI
```bash
curl -L https://app.drosera.io/install | bash
source /root/.bashrc
```
```bash
droseraup
```

---

## 2. Update `drosera.roml`
Replace the previous testnet relay url set in `drosera_rpc` field in `drosera.toml` with relay url.
```bash
cd my-drosera-trap
```
```bash
nano drosera.toml
```
Update the following variables as explained:
* `ethereum_rpc`: Update with Hoodi network rpc. You can usee [Alchemy](https://www.alchemy.com/) or any other third party. To get started, use this: `https://rpc.hoodi.ethpandaops.io`
* `drosera_rpc`: `https://relay.hoodi.drosera.io`
* `eth_chain_id` = `560048`
* `drosera_address` = `"0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"`
* `address`: Remove this line if it exists. It specifies the address of your trap on Holesky. Keeping it when deploying a new trap on Hoodie will cause an error.

Update the trap config with Immortalize Discord Username contract on Hoodi:
* `response_contract`= "0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608"
* `response_function` = "respondWithDiscordName(string)"

Now your `Drosera.toml` file should look something like this:

<img width="833" height="257" alt="image" src="https://github.com/user-attachments/assets/61ff58b4-b331-4d1e-8985-9a690421c2ee" />


---

## 3. Update the Contract
1. Make sure you are in trap directory:
```
cd my-drosera-trap
```

2. Create a new Trap.sol file:
```
nano src/Trap.sol
```

3. Replase the following contract code in it:
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    // Updated response contract address
    address public constant RESPONSE_CONTRACT = 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608;
    string constant discordName = "DISCORD_USERNAME"; // Replace with your Discord username

    function collect() external view returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }

        return (true, abi.encode(name));
    }
}
```

Replace `DISCORD_USERNAME` with your discord username.
To save: `Ctrl+X`, `Y` & `ENTER`

---

### Get Hoodie faucet
Make sure you have ETH on Hoodi testnet network. [Faucet-1](https://hoodi-faucet.pk910.de/)  [Faucet-2](https://faucet.quicknode.com/ethereum/hoodi) [Faucet-3](https://faucet.chainstack.com/hoodi-testnet-faucet)

---

## 4. Re-Apply Drosera Configurations
1. Compile contract:
```bash
forge build
```

2. Test the newtrap before applying:
```bash
drosera dryrun
```

3. Apply the Trap:
```bash
source /root/.bashrc
DROSERA_PRIVATE_KEY=xxx drosera apply
```
* Replace `xxx` with your trap's Private Key.
* Enter `ofc`, when prompted.

<img width="739" height="239" alt="image" src="https://github.com/user-attachments/assets/7f49ca60-f9ff-4316-90f6-f9c763486ad1" />

`address` is your new trap address on Hoodie network. Let's add it to the trap config

---

## 5. Verify Trap's Address
Now add your new Hoodi trap address as `address=""` in `drosera.toml` file:

```bash
cd my-drosera-trap
```
```bash
nano drosera.toml
```
* Add a line starting with `address=""` with its value set to your new trap address on Hoodi from step 4

<img width="852" height="304" alt="image" src="https://github.com/user-attachments/assets/78ed3ae8-700f-4aa6-a6f1-236eaac2e217" />


---

## 6. Open UDP ports
```console
# 1st operator
ufw allow 31313/udp
ufw allow 31314/udp

# 2nd operator
ufw allow 31315/udp
ufw allow 31316/udp
```

---

## 7. Re-Run Operator Node(s)
1. Head to Operator's directory:
```bash
cd ~
```
```bash
cd Drosera-Network
```

2. Stop Operator Nodes:
```bash
docker compose down -v
```

3. Edit `docker-compose.yaml`
```bash
nano docker-compose.yaml
```
* Update `--drosera-address` value for each operator to: `0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D`
* Update `--eth-rpc-url` value for each operator to a Hoodi rpc
  * Create your own **Ethereum Hoodi RPC** in [Alchemy](https://dashboard.alchemy.com/) or [QuickNode](https://dashboard.quicknode.com/).
  * Or use these public rpcs: `https://rpc.hoodi.ethpandaops.io` or `https://0xrpc.io/hoodi`
* Update `--eth-backup-rpc-url` value for each operator to a Hoodi rpc.


4. Restart Nodes:
```bash
docker compose up -d
```

* Node Logs:
```bash
docker compose logs -f
```
* You will get errors initially

![image](https://github.com/user-attachments/assets/c4af432a-cb30-412a-abe4-0e5d0fd5f6ac)

* After a few mintues, you'll get healthy logs:

![image](https://github.com/user-attachments/assets/418229a7-5462-46bd-b81f-a18996a3c822)

Then, Your operators will produce Green Blocks, bro. Congratz.
![image](https://github.com/user-attachments/assets/669b4048-3952-4079-95e1-58dd279e194c)
