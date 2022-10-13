// SPDX License-Identifier: MIT

pragma solidity ^0.8.9;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
// The above is sectio 1, very simple. Solidity version + the two imports just make it much easier to write these contracts.

contract NFT is ERC721, Ownable { //this is contract name (NFT), it is an ERC721 and it is ownable

  using Strings for uint256;

    uint public constant MAX_TOKENS = 10000; //supply
    uint private constant TOKENS_RESERVED = 5; //when smart contract is deplyed, 5 will be minted to my wallet
    uint public price = 100000000000000000; //mint price in wei, actually = 0.01 ETH
    uint256 public constant MAX_MINT_PER_TX = 10; //max amount per wallet to mint = 10

    bool public isSaleActive; //will be true if the sale is live, fasle if not
    uint256 public totalSupply; //dont worry about this
    mapping(address => uint256) private mintedPerWallet; //keps track of how many NFT's each wallet has minted

    string public baseUri;
    string public baseExtesion = ".json";

//Everything above is all variables. If they dont have a value(light freenc colour) e.g isSaleActive it means they will be used later in the contract)
  
   constructor() ERC721("NFT Name", "SYMBOL") { //A constructor will run all the code inside when the contract is deployed. In this case it will create 5 ERC721 tokens with a name and a symbol
        baseUri = "ipfs://xxxxxxxxxxxxxxxxxxxxxxxxxxxxx/"; //This is the baseUri, this will be an IFPS link, only once you have uploaded all your images and json files will you get the IPFS link
        for(uint256 i = 1; i <= TOKENS_RESERVED; ++i) { //this section is a loop on how many tokens are reserved, we have reserved 5 tokens.
            _safeMint(msg.sender, i); 
        }
        totalSupply = TOKENS_RESERVED; //since there will be 5 minted at the end of this, the total supply will be 5
    }

}

    // Everything below are public functions
    function mint(uint256 _numTokens) external payable { //this is the mint function, it takes a number (numTokens)
        require(isSaleActive, "The sale is paused."); //a bunch of require statements, this requires the sale to be live, otherwise it will give the the sale is paused messege
        require(_numTokens <= MAX_MINT_PER_TX, "You cannot mint that many in one transaction."); //this requires the number of tokens you are minting to be less than or equal to max mint pers transaction, in this case 10.
        require(mintedPerWallet[msg.sender] + _numTokens <= MAX_MINT_PER_TX, "You cannot mint that many total."); //the address of whoever is calling the function, if someone tries to mint 10 it will deny you
        uint256 curTotalSupply = totalSupply; //defining current total supply and making it equal to total supply. We can make a temporary variable as the current total supply
        require(curTotalSupply + _numTokens <= MAX_TOKENS, "Exceeds total supply.");// Here we are taking the current total supply + the number of tokens you are trying to mint has to be less than or equal to max tokens
        require(_numTokens * price <= msg.value, "Insufficient funds."); //checks to see if you have the right amount of ethereum to mint. Takes number of tokens you are trying to mint and multiplies it by the price. Must be less than or equal to your amount of eth
//now once the contract has gone through all of these functions to make sure you are elligble to mint then it will loop through all of the tokens and allow you to mint them in your name
        for(uint256 i = 1; i <= _numTokens; ++i) {
            _safeMint(msg.sender, curTotalSupply + 1);
        }
        mintedPerWallet[msg.sender] += _numTokens;//once you have minted this will update with the amount of tokens you have minted
        totalSupply += _numTokens; //the total supply will also update

// Everything below is Owner-only functions
    function flipSaleState() external onlyOwner { //we know this is an onlyowner function beause of the onlyOwner variable on the end
        isSaleActive = !isSaleActive; //if the sale is active it will set it to active, if it is inactive it will set it to inactive. By default this will be inactive, so if you call the function it will be active and people can mint
    }

    function setBaseUri(string memory _baseUri) external onlyOwner { //this function is setBaseUri, it takes a string called baseUri and sets the baseUri variable which we put at the top and changes it to a new one.
        baseUri = _baseUri; //the most common use for this is a late reveal NFT, lets say all our NFTs are minted and we want to change our image to the post reveal image, we would call this function with the new baseUri and then it will change automatically
    }

    function setPrice(uint256 _price) external onlyOwner {
        price = _price;//this will set the price to a new price. We could call this function with a new price and it will change. This is useful if we want to change the rpice during marketing phase, or we have a different price for presale, whitelist, public sale etc.
    }

//when you call the below function (withdrawlAll) it will withdrawl all the eth from the smart contract and split it into (in this case) two wallets
    function withdrawAll() external payable onlyOwner {
        uint256 balance = address(this).balance;
        uint256 balanceOne = balance * 70 / 100;
        uint256 balanceTwo = balance * 30 / 100;
        ( bool transferOne, ) = payable(0x7ceB3cAf7cA83D837F9d04c59f41a92c1dC71C7d).call{value: balanceOne}("");// this wallet will get 70%
        ( bool transferTwo, ) = payable(0x7ceB3cAf7cA83D837F9d04c59f41a92c1dC71C7d).call{value: balanceTwo}("");// this wallet will get 30%
        require(transferOne && transferTwo, "Transfer failed.");
    }
}
