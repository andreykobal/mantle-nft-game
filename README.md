# Mantle NFTVerse: Diamond Quest Adventure

Welcome to the **Mantle NFTVerse: Diamond Quest Adventure** GitHub repository! This repository contains the Unity game project along with the smart contracts used in the game. Diamond Quest Adventure is an exciting 3D game where players use NFT avatars, complete quests, collect diamonds, and explore magical worlds.

## Project Setup

To set up the Diamond Quest Adventure project, please follow these steps:

1. Clone the repository to your local machine using the following command:
   ```
   git clone https://github.com/andreykobal/mantle-nft-game.git
   ```

2. Open the Unity game project that is located in `/unity-project` using Unity Hub or your preferred Unity editor.

3. Install the necessary dependencies and packages required for the project.

4. Configure the Unity project settings as needed and ensure that the project is properly configured for your target platform.

5. Set up the Metamask extension in your preferred browser and create a wallet.

6. Configure the Metamask wallet by connecting it to the Mantle network and obtaining the necessary testnet tokens [Mantle Quick Start](https://docs.mantle.xyz/network/introducing-mantle/quick-start). 

7. Build and run the game to start playing Diamond Quest Adventure!

## Smart Contract Deployment

To deploy the smart contracts using Hardhat, please follow these steps:

* Clone this and run `npm i` in terminal.
* Add .env file with:
```
MNEMONIC=privatekey. not the seedphrase
```
* Edit the deploy script to pass in your name and ticker/uri depending on standard
* Edit the contractUri method in the contract and add your OpenSea collection URI 
* Edit the mint script and add your token uri, contract address and account address of the account you want to mint to.
* Deploy with `npx hardhat run --network mantle scripts/deploy721.js` for ERC-721
* * Deploy with `npx hardhat run --network mantle scripts/deploy1155.js` for ERC-1155
* Mint with `npx hardhat run --network mantle scripts/mint.js`
* Check balance of ERC-1155 with `npx hardhat run --network mantle scripts/balances.js`

Contract code inspired from Opensea: https://github.com/ProjectOpenSea/meta-transactions/blob/main/contracts/ERC721MetaTransactionMaticSample.sol
This is for gas-less transactions when transferring assets. Users dont have to pay that extra gas, and get a better experience.

## Smart Contract Code Highlights

### GameItem.sol

The `GameItem.sol` contract is responsible for managing the 721 non-fungible token (NFT) avatars in Diamond Quest Adventure. Players can purchase avatars and own them using this contract.

Important code snippets from `GameItem.sol`:

- `mintItem` function: This function mints a new avatar NFT with the provided token URI and assigns it to the caller's address.
  ```solidity
  function mintItem(string memory tokenURI) public payable {
      // Mint a new avatar NFT and assign it to the caller's address
      // ...
  }
  ```

### GameItems.sol

The `GameItems.sol` contract handles the 1155 semi-fungible token (SFT) diamonds in Diamond Quest Adventure. Players can collect these diamonds and use them to claim rewards.

Important code snippets from `GameItems.sol`:

- `batchMint` function: This function allows the batch minting of diamond tokens with the provided token URIs and assigns them to the caller's address.
  ```solidity
  function batchMint(string[] memory tokenURIs) public {
      // Batch mint diamond tokens and assign them to the caller's address
      // ...
  }
  ```

- `getTotalBalance` function: This function returns the total balance of diamonds owned by a specific address.
  ```solidity
  function getTotalBalance(address account) public view returns (uint256) {
      // Get the total balance of diamonds owned by the specified address
      // ...
  }
  ```

## Game Code Highlights

Here are some C# code highlights from the Diamond Quest Adventure project:

### MintNFT.cs

```csharp
async public void mintItem(int avatarIndex)
{
    string chainId = "5001";
    var tokenURI = "https://bafkreiczapdwomdlotjqt4yaojyizlgarn4kq57smi3ptkwn5lug5yz7yu.ipfs.nftstorage.link/";

    string contractAbi = "";
    string contractAddress = contractAddresses[avatarIndex];
    string method = "mintItem";
    
    var provider = new JsonRpcProvider("https://rpc.testnet.mantle.xyz/");
    var contract = new Contract(contractAbi, contractAddress, provider);
    Debug.Log("Contract: " + contract);
    
    var calldata = contract.Calldata(method, new object[]
    {
        tokenURI
    });
    
    string response = await Web3Wallet.SendTransaction(chainId, contractAddress, "500000000000000000", calldata, "", "");

    Debug.LogError(response);
    
    StartCoroutine(WaitAndCheckBalance(avatarIndex));
}
```

### CheckNFTOwnership.cs

```csharp
async public void CheckNFTBalance(string contractAddress, int avatarIndex)
{
    string contractAbi = "";
    string method = "getBalance";

    var provider = new JsonRpcProvider("https://rpc.testnet.mantle.xyz/");
    var contract = new Contract(contractAbi, contractAddress, provider);
    Debug.Log("Contract: " + contract);


    // Call the getBalance method
    var walletAddress = PlayerPrefs.GetString("Account");
    Debug.Log("Wallet address is: " + walletAddress);
    var calldata = await contract.Call(method, new object[]
    {
        // Pass the wallet address for which you want to retrieve the balance
        walletAddress
    });
    avatarBalances[avatarIndex] = int.Parse(calldata[0].ToString());
    print("ERC-721 Token Balance for avatar " + (avatarIndex + 1) + ": " + avatarBalances[avatarIndex]);

    string buttonText = avatarBalances[avatarIndex] > 0 ? "Select" : "Buy 0.5 $MNT";
    ui.UpdateButtonLabel(avatarIndex, buttonText);
}
```

### Check1155Balance.cs

```csharp
async public void Check1155TotalBalance()
{
    string contractAbi = "";
    string contractAddress = "0x6721De8B1865A6cD98C64165305611B1f28B95e4";
    string method = "getTotalBalance";
    
    var walletAddress = PlayerPrefs.GetString("Account");
    var provider = new JsonRpcProvider("https://rpc.testnet.mantle.xyz/");
    var contract = new Contract(contractAbi, contractAddress, provider);
    var calldata = await contract.Call(method, new object[]
    {
        // Pass the account address as the parameter
        walletAddress
    });
    var totalDiamonds = calldata[0];
    
    print("Contract 1155 Balance (Diamonds) Total: " + totalDiamonds);
    
    diamondsTotalLabel = uiDocument.rootVisualElement.Q<Label>("DiamondsTotal");
    diamondsTotalLabel.text = totalDiamonds.ToString();
}
```

### MintDiamonds.cs

```csharp
private async void Start()
{
    // Find the "Claim" button in the UI document
    Button claimButton = GetComponent<UIDocument>().rootVisualElement.Q<Button>("ClaimButton");
    claimElement = GetComponent<UIDocument>().rootVisualElement.Q("Claim");

    // Add a click event handler to the "Claim" button
    claimButton.clicked += Mint1155Diamonds;
}

async public void Mint1155Diamonds()
{
    string chainId = "5001";
    string contractAddress = "0x6721De8B1865A6cD98C64165305611B1f28B95e4";
    string value = "0";
    string abi = "";
    string method = "batchMint";
    string gasLimit = "";
    string gasPrice = "";
    string[] uri = { "https://bafkreiczapdwomdlotjqt4yaojyizlgarn4kq57smi3ptkwn5lug5yz7yu.ipfs.nftstorage.link/" };
    var contract = new Contract(abi, contractAddress);
    var calldata = contract.Calldata(method, new object[]
    {
        uri
    });

    string response = await Web3Wallet.SendTransaction(chainId, contractAddress, value, calldata, gasLimit, gasPrice);
    print(response);

    // Check if there is a response
    if (!string.IsNullOrEmpty(response))
    {
        claimElement.visible = false;
    }
}
```

## Contributing

Thank you for your interest in contributing

 to Diamond Quest Adventure! If you would like to contribute, please follow these guidelines:

- Fork the repository and create a new branch for your contributions.
- Make your changes and ensure that the code adheres to the project's coding style and conventions.
- Test your changes thoroughly.
- Submit a pull request describing your changes and the rationale behind them.

We appreciate your contributions to make Diamond Quest Adventure an even more exciting game!

## License

This project is licensed under the [MIT License](LICENSE). Feel free to use, modify, and distribute the code for personal or commercial purposes.

## Acknowledgements

We would like to express our gratitude to the open-source community for their valuable contributions and the resources that helped in the development of Diamond Quest Adventure.

If you have any questions or encounter any issues, please don't hesitate to reach out. Enjoy playing Diamond Quest Adventure!

*Note: The above readme.md is a fictional representation based on the provided information. Please customize it according to your project's requirements and provide accurate and detailed instructions.*
