# BEP-66: Mini-BEP2 Tokens

- [BEP-66: Mini-BEP2 Tokens](#bep-66-mini-bep2-tokens)
  - [1.  Summary](#1--summary)
  - [2.  Abstract](#2--abstract)
  - [3.  Status](#3--status)
  - [4.  Motivation](#4--motivation)
  - [5.  Specification](#5--specification)
    - [5.1 Mini-BEP2 Token on Binance Chain](#51-mini-bep2-token-on-binance-chain)
    - [5.2 Mini-BEP2 Token Properties](#52-mini-bep2-token-properties)
    - [5.3 Token Management Operation](#53-token-management-operation)
      - [5.3.1 Issue Mini-BEP2 Tokens](#531-issue-mini-bep2-tokens)
      - [5.3.2 Transfer Tokens](#532-transfer-tokens)
      - [5.3.3  Freeze Tokens](#533-freeze-tokens)
      - [5.3.4  Unfreeze Tokens](#534-unfreeze-tokens)
      - [5.3.5 Mint Tokens](#535-mint-tokens)
      - [5.3.6 Burn Tokens](#536-burn-tokens)
      - [5.3.7 Set Token URI](#537-set-token-uri)
     - [5.4 Mini-BEP2 Token Trading](#54-mini-bep2-token-trading)
          - [5.4.1 List](#541-list)
          - [5.4.2 Order placement](#542-order-placement)
          - [5.4.3 Match](#543-match)
          - [5.4.4 Delist](#544-delist)
  - [6. License](#6-license)
  
## 1.  Summary

This BEP describes a proposal for Mini-BEP2 token management on the Binance Chain.

## 2.  Abstract

BEP-66 Proposal describes a common set of rules for Mini-BEP2 token management within the Binance Chain ecosystem. It introduces the following details of a token on Binance Chain:

- What information makes a Mini-BEP2 token on Binance Chain
- What actions can be performed on a Mini-BEP2 token on Binance Chain
- How Mini-BEP2 token trading is different from BEP2 token trading:

## 3.  Status

Draft

## 4.  Motivation

Similar to SME board in the traditional stock markets, the BEP-66 Proposal is to benefit the below scenarios:

- small enterprise tokens of utility and/or shares
- point system
- IP tokens

Different from the normal BEP2, using a mini-token system is good at:

- Very cheap (much less than BEP2) to issue tokens.
- Easy and very cheap to list tokens, no need votes.

## 5.  Specification

### 5.1 Mini-BEP2 Token on Binance Chain
   A new module “miniToken” should be created for Mini-BEP2 token. The Mini-BEP2 token should be stored in an independent database from BEP2 token, and new transaction types should be created for Mini-BEP2 token specifically. The minimum transaction amount should be 1e8. Besides, the issue/listing fees are much cheaper than BEP2 token and the Mini-BEP2 token can be listed by the issuer directly without voting. 

### 5.2 Mini-BEP2 Token Properties
- Source Address: Source Address is the owner of the issued token.
- Token Name: Token Name represents the long name of the token, limited to 32 unicode - e.g. "ABCcoin".
- Symbol: Symbol is the identifier of the newly issued token - e.g. “ABCcoin.M-6YZ”.
- Total Supply: Total supply will be the total number of issued tokens.
- Mintable: Mintable means whether this token can be minted in the future, which would increase the total supply of the token
- Token URI: A distinct Uniform Resource Identifier (URI) for the token. 
- Supply Range: 1 = [1-10K] ; 2=(10K, 100K]. Range 2 will charge more than Range 1 for list and issue. 

### 5.3 Token Management Operation

#### 5.3.1 Issue Mini-BEP2 Tokens
Issuing token is to create a new Mini-BEP2 token on Binance Chain. The new Mini-BEP2 token name should be suffixed with “.M”. The issue fee is different from the BEP2 token’s.  
**Data Structure for Issue Operation**: A data structure is needed to represent the new Mini-BEP2 token:

| **Field**    | **Type** | **Description**                                              |
| :------------ | :-------- | :------------------------------------------------------------ |
| Name         | string   | Name of the newly issued asset, limited to 32 unicode characters,  e.g. "ABCcoin" |
| Symbol       | string   | The length of the string for representing this asset is between 3 and 8 alphanumeric characters and is case insensitive. The symbol will be suffixed with “.M” to distinguish from BEP2 token and followed by the first 3 bytes of the issue transaction hash to remove a constraint of requiring unique token names. |
| TokenURI     | URI string| Optional. A distinct Uniform Resource Identifier (URI) for the token. The URI may point to a JSON file that conforms to the "Mini-BEP2 Metadata JSON Schema". The schema is optional. |
| TotalSupply  | int64    | The total supply for this token can have a maximum of 8 digits of decimal and is boosted by 1e8 in order to store as int64. The amount before boosting should not exceed upper bound of supply range. |
| SupplyRange  | int      | 1 = [1-10K] ; 2= (10K, 100K] |
| Owner        | Address  | The initial issuer of this token, the BNB balance of issuer should be more than the fee for issuing tokens |
| Mintable     | Boolean  | Whether this token could be minted(increased) after the initial issuing |

The data in all the above fields are not changeable after the Issue Transaction, except “Total Supply” can be changed via “Mint” or “Burn” operations. And TokenURI can be changed by “SetTokenURI” Transaction.  

**Symbol Convention:**

[Symbol][B]-[Suffix]

Explanations: “.M” is to distinguish this Mini token from normal BEP2 token. Suffix is the first 3 bytes of the issue transaction hash. It helps to remove the constraint of requiring unique token names.  

**Mini-BEP2 Metadata JSON Schema:**
```json
{
  "name": "Mini Token Metadata",
  "description": "Metadata description for the Mini Token", 
  "external_url": "https://example.com/token", 
  "image": "https://example.com/token/1.png", 
  "attributes": [
    {
      "name": "custom field", 
      "value": "custom value"
    }, 
    ...
  ]
}

```

**Issue Process:**

- Issuer signed an issue transaction and make it broadcasted to one of Binance Chain nodes
- This Binance Chain node will check this transaction. If there is no error, then this transaction will be broadcasted to other Binance Chain nodes
- Issue transaction is committed on the blockchain by block proposer
- Validators will verify the constraints on total supply and symbol and deduct the fee from issuer’s account
- New token’s symbol is generated based on the transaction hash. It is added to the issuer’s address and token info is saved on the Binance Chain  

#### 5.3.2 Transfer Tokens
There is a new transaction type “miniTokens/Send”. The message structure and transaction process are the same as BEP2 5.3.2 Transfer Tokens. The difference from BEP2 transfer is that the Mini-BEP2 transfer amount should be larger than or equal to 1e8, unless the sender sends all of the free Mini-BEP2 token in his account.

#### 5.3.3 Freeze Tokens
There is a new transaction type  “miniTokens/Freeze”. The message structure and transaction process are the same as BEP2 5.3.3 Freeze Tokens, except that the amount should be larger than or equal to 1e8, or equal to the free account balance.

#### 5.3.4 Unfreeze Tokens
There is a new transaction type “miniTokens/Unfreeze”.  The message structure and transaction process are the same as BEP2 5.3.4 Unfreeze Tokens, except that the amount should be larger than or equal to 1e8, or equal to the frozen account balance.

#### 5.3.5 Mint Tokens
There is a new transaction type “miniTokens/Mint”.  The message structure and transaction process are the same as BEP2 5.3.5 Mint Tokens, except that the amount should be larger than or equal to 1e8.

#### 5.3.6 Burn Tokens
There is a new transaction type “miniTokens/Burn”.  The message structure and transaction process are the same as BEP2 5.3.6 Burn Tokens. The difference from BEP2 burn is that the Mini-BEP2 burn amount should be larger than or equal to 1e8, or equal to the free account balance.

#### 5.3.7 Set Token URI
The “miniTokens/SetURI” transaction is to change the value of “TokenURI”. Only token issuer can send this transaction.  
**Data structure:**

| **Field**    | **Type** | **Description**                                              |
| :------------ | :-------- | :------------------------------------------------------------ |
| Symbol         | string   | the Mini BEP2 token |
| TokenURI | URI string | A distinct Uniform Resource Identifier (URI) for the token. The URI may point to a JSON file that conforms to the "Mini-BEP2 Metadata JSON Schema".|


### 5.4 Mini-BEP2 Token Trading

#### 5.4.1 List

The list process is different from BEP2 token list. Mini-BEP2 token issuer can list the Mini-BEP2 token without voting. The Mini-BEP2 token can only be listed against BNB or BUSD. The Mini-BEP2 token cannot be listed as a quote symbol.  
There is a new transaction type “ListMini”. 
**The following parameters are required for the transaction:**

| **Field**    | **Type** | **Description**                                              |
| :------------ | :-------- | :------------------------------------------------------------ |
| base-asset-symbol | string | the Mini BEP2 token to list|
|quote-asset-symbol| string|only support BNB and BUSD as quote asset|
|init-price|int64|the initial price for your asset, it is boosted by 1e8|
|proposal-id|string|this corresponds to the Mini-BEP2 listing proposal that is passed|
|from|Bech32_address|this address should be the issuer of base asset|

#### 5.4.2 Order placement
The minimum amount of the Mini-BEP2 token should be larger than or equal to 1e8. The only exceptional case is that user sells all of the free Mini-BEP2 token in his account.
#### 5.4.3 Match
Unlike BEP2 token, not every Mini-BEP2 token pair will be matched in every block. But user can still expect a fast match (less than 10 seconds). The most active pairs  will be picked for match in each new block.
#### 5.4.4 Delist
Delist a trading pair of Mini-BEP2 token is the same as BEP2 delist. Please refer to BEP-6.

## 6. License

The content is licensed under [CC0](https://creativecommons.org/publicdomain/zero/1.0/).



 
