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
* `ethereum_rpc`: Update with Hoodi network rpc. You can use [Alchemy](https://www.alchemy.com/) or [Ankr](https://www.ankr.com/rpc/?utm_referral=LqL9Sv86Te) (Paid) or any other third party. To get started, use this: `https://rpc.hoodi.ethpandaops.io`
* `drosera_rpc`: `https://relay.hoodi.drosera.io`
* `eth_chain_id` = `560048`
* `drosera_address` = `"0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"`
* `address`: Remove this line if it exists. It specifies the address of your trap on Holesky. Keeping it when deploying a new trap on Hoodie will cause an error.

Update the trap config with Immortalize Discord Username contract on Hoodi:
* `response_contract`= `"0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608"`
* `response_function` = `"respondWithDiscordName(string)"`

Now your `Drosera.toml` file should look something like this:

<img width="855" height="311" alt="image" src="https://github.com/user-attachments/assets/ae53c341-50ec-43fa-ac66-96d81cc570d4" />

---

## 3. Update the Contract
The default contract of the Drosera is a simple HelloWorld, but we deploy a new one to immortalize our Discord usernames and earn **Cadet** role in Discord. Even if you did this before, replace the new code in the next steps because of a minor update in the code.

### 1. Make sure you are in trap directory:
```
cd my-drosera-trap
```

### 2. Create a new Trap.sol file:
```
nano src/Trap.sol
```

### 3. Replase the following contract code in it:
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
### 1. Compile contract:
```bash
forge build
```

### 2. Test the newtrap before applying:
```bash
drosera dryrun
```

### 3. Apply the Trap:
```bash
source /root/.bashrc
DROSERA_PRIVATE_KEY=xxx drosera apply
```
* Replace `xxx` with your trap's Private Key.
* Enter `ofc`, when prompted.

<img width="739" height="239" alt="image" src="https://github.com/user-attachments/assets/7f49ca60-f9ff-4316-90f6-f9c763486ad1" />

* `address` is your new trap address on Hoodie network. Let's add it to the trap config.
* You can check your Trap address on Hoodi testnet here: https://app.drosera.io/


---

## 5. Add Trap's Address to your config
Now add your new Hoodi trap address as `address=""` in `drosera.toml` file:

```bash
cd my-drosera-trap
```
```bash
nano drosera.toml
```
* Add a line starting with `address=""` with its value set to your new trap address on Hoodi from step 4

<img width="887" height="300" alt="image" src="https://github.com/user-attachments/assets/89b36df6-b1a4-449b-bc54-368162357909" />


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
### 1. Head to Operator's directory:
```bash
cd ~
```
```bash
cd Drosera-Network
```

### 2. Stop Operator(s):
```bash
docker compose down -v
```

### 3. Edit `docker-compose.yaml`
```bash
nano docker-compose.yaml
```
* Update `--drosera-address` value for each operator to: `0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D`
* Update `--eth-rpc-url` value for each operator to a Hoodi rpc
  * Create your own **Ethereum Hoodi RPC** in [Alchemy](https://dashboard.alchemy.com/) or [QuickNode](https://dashboard.quicknode.com/) or [Ankr](https://www.ankr.com/rpc/?utm_referral=LqL9Sv86Te) (Paid)
  * Or use these public rpcs: `https://rpc.hoodi.ethpandaops.io` or `https://0xrpc.io/hoodi`
* Update `--eth-backup-rpc-url` value for each operator to a Hoodi rpc.


### 4. Register Operator(s)
```
source /root/.bashrc

drosera-operator register --eth-rpc-url https://rpc.hoodi.ethpandaops.io --eth-private-key PV_KEY --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```
* Replace `PV_KEY` with your operator privatekey
* If you have more operators, execute the command for each operator

<img width="675" height="53" alt="image" src="https://github.com/user-attachments/assets/20aa3957-1cfb-4161-a5da-69dc3368ceec" />


### 5. Restart Operator(s):
```bash
docker compose up -d
```

### 6. Node Logs:
```bash
docker compose logs -f
```
* You will get `Operator Node successfully spawned!`. Now your oeprator(s) have to opt-in to the trap

<img width="1387" height="608" alt="image" src="https://github.com/user-attachments/assets/e2347d5d-64f6-4f7b-935a-0ca4a9073fa7" />


### 7. Opt-in Operator(s) to your Trap:
Connect your operator wallet to the [Dashboard](https://app.drosera.io/), Find your Trap address, click on `Opti in` to connect your operator to the Trap

![image](https://github.com/user-attachments/assets/5189b5cb-cb46-4d10-938a-33f71951dfc2)


Now refresh the Trap page, Green blocks incoming bro...

<img width="1785" height="563" alt="image" src="https://github.com/user-attachments/assets/22b4640b-e481-4cf3-89d8-ae2c25228d2d" />

---

## 8. Verify Trap can respond
After the trap is deployed and you got green blocks in the dashboard, we can check if the user has responded by calling the `isResponder` function on the response contract.
```bash
source /root/.bashrc
cast call 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608 "isResponder(address)(bool)" OWNER_ADDRESS --rpc-url https://rpc.hoodi.ethpandaops.io
```
* Replace `OWNER_ADDRESS` with your Trap's owner address. (Your main address that has deployed the Trap's contract)
* If you receive `true` as a response, it means you have successfully completed all the steps.

![image](https://github.com/user-attachments/assets/b6f89508-1ce4-46d6-8dcb-685ae7063d07)

* It may take a few minutes to successfully receive `true` as a response

### 6. View the List of submitted Discord Names
```bash
source /root/.bashrc
cast call 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608 "getDiscordNamesBatch(uint256,uint256)(string[])" 0 2000 --rpc-url https://rpc.hoodi.ethpandaops.io
```

