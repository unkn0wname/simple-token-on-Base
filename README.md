# Token On Base

A guide to deploy a token to Base with some simple features: mint, burn, ...

## Prerequisites

-  Node.js (v18 or higher)
-  EVM compatible wallet
-  ETH for gas fees
-  Text or code editor

## Getting Started

1. Install Foundry:
```bash
curl -L https://foundry.paradigm.xyz | bash
export PATH="$HOME/.foundry/bin:$PATH"
foundryup
```
2. Create root folder:

This folder will contain all the required code and files for the token
```bash
forge init your_folder
```
Delete all the template files
```bash
cd your_folder
rm src/Counter.sol script/Counter.s.sol test/Counter.t.sol
```
3. Install OpenZeppelin contracts
```bash
install OpenZeppelin/openzeppelin-contracts
```

## Configuring The Code
1. Create the token contract

In `your_folder/src`, create a file named `MyToken.sol` and write in
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title MyToken
 * @dev ERC-20 token with minting capabilities and supply cap
 */
contract MyToken is ERC20, Ownable {
    // Maximum number of tokens that can ever exist
    uint256 public constant MAX_SUPPLY = 1_000_000_000 * 10**18; // 1 billion tokens
    
    constructor(
        string memory name,
        string memory symbol,
        uint256 initialSupply,
        address initialOwner
    ) ERC20(name, symbol) Ownable(initialOwner) {
        require(initialSupply <= MAX_SUPPLY, "Initial supply exceeds max supply");
        // Mint initial supply to the contract deployer
        _mint(initialOwner, initialSupply);
    }
    
    /**
     * @dev Mint new tokens (only contract owner can call this)
     * @param to Address to mint tokens to
     * @param amount Amount of tokens to mint
     */
    function mint(address to, uint256 amount) public onlyOwner {
        require(totalSupply() + amount <= MAX_SUPPLY, "Minting would exceed max supply");
        _mint(to, amount);
    }
    
    /**
     * @dev Burn tokens from caller's balance
     * @param amount Amount of tokens to burn
     */
    function burn(uint256 amount) public {
        _burn(msg.sender, amount);
    }
}
```
- `1_000_000_000`: total supply of the token

2. Create the deployment script

In `your_folder/script`, create a file named `DeployToken.s.sol` and write in
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {Script, console} from "forge-std/Script.sol";
import {MyToken} from "../src/MyToken.sol";

contract DeployToken is Script {
    function run() external {
        // Load deployer's private key from environment variables
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        address deployerAddress = vm.addr(deployerPrivateKey);
        
        // Token configuration parameters
        string memory name = "My Token";
        string memory symbol = "MTK";
        uint256 initialSupply = 100_000_000 * 10**18; // 100 million tokens
        
        // Start broadcasting transactions
        vm.startBroadcast(deployerPrivateKey);
        
        // Deploy the token contract
        MyToken token = new MyToken(
            name,
            symbol,
            initialSupply,
            deployerAddress
        );
        
        // Stop broadcasting transactions
        vm.stopBroadcast();
        
        // Log deployment information
        console.log("Token deployed to:", address(token));
        console.log("Token name:", token.name());
        console.log("Token symbol:", token.symbol());
        console.log("Initial supply:", token.totalSupply());
        console.log("Deployer balance:", token.balanceOf(deployerAddress));
    }
}
```
- `My Token`: token's name
- `MTK`: token's symbol
- `1_000_000`: amount of token's initial supply (all of them will be in your wallet)

3. Configuring the environment

In `your_folder`, create a file named `.env` and write in
```
PRIVATE_KEY=your_private_key_here
BASE_SEPOLIA_RPC_URL=https://sepolia.base.org
BASE_MAINNET_RPC_URL=https://mainnet.base.org
```
- `PRIVATE_KEY`: your wallet's private key

In `your_folder`, open a file named `foundry.tomb` and update it
```
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
remappings = ["@openzeppelin/=lib/openzeppelin-contracts/"]

[rpc_endpoints]
base_sepolia = "${BASE_SEPOLIA_RPC_URL}"
base_mainnet = "${BASE_MAINNET_RPC_URL}"

[etherscan]
base_sepolia = { key = "${BASESCAN_API_KEY}", url = "https://api-sepolia.basescan.org/api" }
base = { key = "${BASESCAN_API_KEY}", url = "https://api.basescan.org/api" }
```

## Deployment And Verification

NOTE: I recommend deploying and verifying it on Base Testnet before Base Mainnet for testing

1. Deployment
```bash
source .env
forge script script/DeployToken.s.sol:DeployToken \--rpc-url base_mainnet \--broadcast
```
- You need to change `base_mainnet` to `base_sepolia` to deploy to Base Testnet

After deploying, a file will appear named `run-latest.json` in `your_folder/broadcast`. Find a line like `"contractAddress": "0x...",` and copy the `0x...`

2. Verification

Verifying that you deploy this contract to write in this contract
```bash
forge verify-contract --watch --chain base contract_address src/MyToken.sol:MyToken --verifier etherscan --etherscan-api-key YourApiKeyToken
```
- `contract_address`: paste the `0x...` you copied before
- `YourApiKeyToken`: visit https://etherscan.io/apidashboard to grab yours and fill in

- You need to change `base` to `base-sepolia` to deploy to Base Testnet

### You just deployed a token to Base

## License

This project is licensed under the MIT License - see the LICENSE file for details.
