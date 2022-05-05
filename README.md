// SPDX-License-Identifier: MIT

/*
FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK               FKFKFKFKFKFKFKFKFKFKFK     FKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKF
FKFKFKFKFKFKFK               FKFKFKFKFKFKFKFKFKFKFK     FKFKFKFK    FKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK     FKFKFK    FKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK     FKFK    FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK             FKFKFKFKFKFKFKFKFKFKFKFK     FK    FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK             FKFKFKFKFKFKFKFKFKFKFKFK         FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK     FK    FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK     FKFK    FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK     FKFKFK    FKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK     FKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKF
FKFKFKFKFKFKFK     FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK     FKFKFKFKFK      FKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
FKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFKFK
*/

pragma solidity >=0.7.0 <0.9.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

 //Koleksiyon adı ↓ 
contract NFTCOLLECTION is ERC721, Ownable {
  using Strings for uint256;
  using Counters for Counters.Counter;

  Counters.Counter private supply;
 //NFT Görsellerinin ana metadata CID kodu  ↓ (uriPreFix)
  string public uriPrefix = "ipfs://___CID-Metadata___/";
  string public uriSuffix = ".json";
  string public hiddenMetadataUri;
 
  uint256 public cost = 0.00001 ether;  // NFT Koleksiyon Eth Değeri
  uint256 public maxSupply = 250;  // NFT Koleksiyon Parça Miktarı
  uint256 public maxMintAmountPerTx = 50;  // NFT Koleksiyonun Cüzdan Başı Mint Miktarı

  bool public paused = false;  // NFT Sözleşme Mintlemeye Açık (false), Mintlemeye kapalı (true)
  bool public revealed = true;  // NFT Sözleşmedeki parçalar görünür (true), parçalar görünürlüğü kapalı (false)

 // NFT: NAME-SYMBOL YAZILACAK. ↓ 
  constructor() ERC721("NAME", "SYMBOL") {
    setHiddenMetadataUri("ipfs://Qmb9C4PVGNbZfmR1owya2QmcETiHT29uJRUomgBEdLqVtb/hidden.png");  // NFT GİZLİ (hidden.json) METADATA
  }

  modifier mintCompliance(uint256 _mintAmount) {
    require(_mintAmount > 0 && _mintAmount <= maxMintAmountPerTx, "Invalid mint amount!");
    require(supply.current() + _mintAmount <= maxSupply, "Max supply exceeded!");
    _;
  }

  function totalSupply() public view returns (uint256) {
    return supply.current();
  }

  function mint(uint256 _mintAmount) public payable mintCompliance(_mintAmount) {
    require(!paused, "The contract is paused!");
    require(msg.value >= cost * _mintAmount, "Insufficient funds!");

    _mintLoop(msg.sender, _mintAmount);
  }
  
  function mintForAddress(uint256 _mintAmount, address _receiver) public mintCompliance(_mintAmount) onlyOwner {
    _mintLoop(_receiver, _mintAmount);
  }

  function walletOfOwner(address _owner)
    public
    view
    returns (uint256[] memory)
  {
    uint256 ownerTokenCount = balanceOf(_owner);
    uint256[] memory ownedTokenIds = new uint256[](ownerTokenCount);
    uint256 currentTokenId = 1;
    uint256 ownedTokenIndex = 0;

    while (ownedTokenIndex < ownerTokenCount && currentTokenId <= maxSupply) {
      address currentTokenOwner = ownerOf(currentTokenId);

      if (currentTokenOwner == _owner) {
        ownedTokenIds[ownedTokenIndex] = currentTokenId;

        ownedTokenIndex++;
      }

      currentTokenId++;
    }

    return ownedTokenIds;
  }

  function tokenURI(uint256 _tokenId)
    public
    view
    virtual
    override
    returns (string memory)
  {
    require(
      _exists(_tokenId),
      "ERC721Metadata: URI query for nonexistent token"
    );

    if (revealed == false) {
      return hiddenMetadataUri;
    }

    string memory currentBaseURI = _baseURI();
    return bytes(currentBaseURI).length > 0
        ? string(abi.encodePacked(currentBaseURI, _tokenId.toString(), uriSuffix))
        : "";
  }

  function setRevealed(bool _state) public onlyOwner {
    revealed = _state;
  }

  function setCost(uint256 _cost) public onlyOwner {
    cost = _cost;
  }

  function setMaxMintAmountPerTx(uint256 _maxMintAmountPerTx) public onlyOwner {
    maxMintAmountPerTx = _maxMintAmountPerTx;
  }

  function setHiddenMetadataUri(string memory _hiddenMetadataUri) public onlyOwner {
    hiddenMetadataUri = _hiddenMetadataUri;
  }

  function setUriPrefix(string memory _uriPrefix) public onlyOwner {
    uriPrefix = _uriPrefix;
  }

  function setUriSuffix(string memory _uriSuffix) public onlyOwner {
    uriSuffix = _uriSuffix;
  }

  function setPaused(bool _state) public onlyOwner {
    paused = _state;
  }

  // NFT Sözleşmeden Para Çekme ↓ (sahiplik bölümü)
  function withdraw() public onlyOwner {
 // =============================================================================   |PAY VERİLECEKSE BU KSIIM DURACAK|
 
 // Bu koleksiyon için pay vereceğin cüzdan   ↓  ↓  ↓  ↓    // Bu kısımda verilecek payın yüzdelik kısmı  ↓ 
    (bool hs, ) = payable(0x2EB46561F11F1f1fF2DaEC1D9d256149022A3a53).call{value: address(this).balance * 5 / 100}("");
    require(hs);
 // =============================================================================   |PAY VERİLMEYECEKSE ÜST KISMI SİL|
   
    (bool os, ) = payable(owner()).call{value: address(this).balance}("");
    require(os);
  }
 // =============================================================================
  function _mintLoop(address _receiver, uint256 _mintAmount) internal {
    for (uint256 i = 0; i < _mintAmount; i++) {
      supply.increment();
      _safeMint(_receiver, supply.current());
    }
  }

  function _baseURI() internal view virtual override returns (string memory) {
    return uriPrefix;
  }
}
