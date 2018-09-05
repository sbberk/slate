---
title: Tradeshift Makerdao Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  # - errors

search: true
---

# Getting Started

## Get the code and install dependencies
`git clone https://github.com/Tradeshift/ts-cash-js `

`git checkout develop-clean   `

`curl https://dapp.tools/install | sh  `

`git submodule update --init --recursive `

`npm install`

Now install the private npm package for Tradeshift
In the consumer project folder:  
`npm login your_username  `

`npm install '@makerdao/ts-cash-js' `

Now you can run the tests. In the library folder:  

`npm run test `

This will start a testnet running locally using ganache-cli

# Code Example

Here a Maker object is instantiated, and a FactorService object using the invoiceFactoring service is created. This object acts as a bridge for function calls to the Ethereum network.

```javascript 
// in the index.js file
import Maker, { InvoiceFactoringService, InvoiceMarketService } from '@makerdao/ts-cash-js';

const maker = await Maker.create('test', {
    additionalServices: [
      'invoiceFactoring',
      'invoiceMarket'
    ],
    invoiceFactoring: [InvoiceFactoringService],
    invoiceMarket: [InvoiceMarketService],
    … other settings … 
  });

Const factorService = maker.service('invoiceFactoring');
Get address and call
// then run node ./index.js 
```

# How Maker is Initialized

The Maker object is the first object instantiated, and is the pathway to instantiate all other services. 

```javascript
Import Maker, { InvoiceFactoringService, InvoiceMarketService, contracts, networks } from ‘@makerdao/ts-cash-js’;
  const mapping = networks.find(m => m.name === 'test');
  const maker = Maker.create('test', {
    additionalServices: [
      'invoiceFactoring',
      'invoiceMarket'
    ],
    invoiceFactoring: [InvoiceFactoringService],
    invoiceMarket: [InvoiceMarketService],
    smartContract: {
      addContracts: {
        [contracts.SAI]: {
          address: mapping.addresses.SAI[0].address,
          abi: mapping.addresses.SAI[0].abi
        },
        [contracts.TS_AUTH]: {
          address: mapping.addresses.TS_AUTH[0].address,
          abi: mapping.addresses.TS_AUTH[0].abi
        },
        [contracts.TS_ADMIN]: {
          address: mapping.addresses.TS_ADMIN[0].address,
          abi: mapping.addresses.TS_ADMIN[0].abi
        },
        [contracts.TS_INVESTOR]: {
          address: mapping.addresses.TS_INVESTOR[0].address,
          abi: mapping.addresses.TS_INVESTOR[0].abi
        }
      }
    }
  }); 
  ```


# Use Case Scenarios

 > The same private key should be used for all use case scenarios.

## Initial Contract Setup
User: Tradeshift
Assumptions: Maker object constructed (documentation), imported from ts-cash-js
Conventions
Auth = TradeshiftAuthority 
User = TradeshiftUsers
Admin = TradeshiftAdmin
Investor = TradeshiftInvestor
 
```javascript
//Deploy contract 
Auth( )
// Deploy contract
Admin(Auth address)
//Deploy contract 
Investor( )
//Call contract function - 
Auth.setOwner(Admin address)
//Call contract function - 
Admin.setTradeshiftApproved(Investor, true)
//OR
//Approve investor contract
await maker.service(‘invoiceFactoring’).approveInvestorContract( ... )
```


