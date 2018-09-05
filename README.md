import Maker, {InvoiceFactoringService, InvoiceMarketService, contracts, networks } from ‘@makerdao/ts-cash-js’;
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

//Starts here https://makerdao.com/documentation/#authenticate and follows maker documentation
//authenticate
await maker.authenticate();


//service
factorService = maker.service('invoiceFactoring');
//OR
marketService = maker.service('invoiceMarket');


//factorInvoice object
const faceValue = 19.6; // principal value of invoice in DAI (value of invoice when fully paid)
const discountedValue = 0.75; //75% of principal value or 25% discount on price expected return of invoice
//returns promise(resolves to a FactoredInvoice object)
const newFactoredInvoice = await factorService.factorInvoice(faceValue,discountedValue);

//getFactoredInvoice, invoiceId = address: string (e.g. "0x65G.....4D46")
const FactoredInvoice = await getFactoredInvoice(invoiceId);


//Note: tab = subsections
//FactoredInvoice Object
//getId, returns a string (invoice token address)
const invoiceId = await FactoredInvoice.getId();

//getTokenSupply - total supply of invoiceToken
const supply = await FactoredInvoice.getTokenSupply();

//getTokenBalance - addresses' balance of invoiceTokens
const tokenAddress = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910'; //dummy address
const balance = await FactoredInvoice.getTokenBalance(tokenAddress);

//getIOUContractAddress(), returns string
await iou = await FactoredInvoice.getIOUContractAddress()

//getSaleContractAddress, returns string
await sale = await FactoredInvoice.getSaleContractAddress()

// from TS_Admin perspective
  //settle
    //settle - autoprovision active, optional settlement amount
    await FactoredInvoice.settle(true);

    //settle - optional settlement amount in DAI
    await FactoredInvoice.settle(true, 20);

    //settle - autoprovision inactive (must send funds to TS_Admin if has insufficient funds)
    await FactoredInvoice.settle(false);

  //sweep
  await FactoredInvoice.sweep();

// from TS_Investor Perspective
  //buy
  await FactoredInvoice.buy();

  //redeem
  await FactoredInvoice.redeem();





// InvoiceFactoringService -> TS_Admin perspective
  //approveInvestorContract
  await factorService.approveInvestorContract(investorAddress,true);

  //getTSAuthOwner, returns
  const TSAuthOwnerAddress = factorService.getTSAuthOwner();

  //isUserApproved, checks if user is TSApproved returns a boolean
  const address = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910'; //dummy address
  const bool = factorService.isUserApproved(address);

  //checkInvoiceToken, checks if address is a tokenAddress, returns a boolean
  const tokenAddress = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910'; //dummy address
  const bool = factorService.checkInvoiceToken(tokenAddress);

  //provision - send DAI from msg.sender to some destination
  const amount = 10; // amount of dai to send to an address
  const TSAdminAddress = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910'; //dummy address
  await factorService.provision(TSAdminAddress, 10);

  //withdraws DAI from TSAdmin to destination
    //withdraw all
    const destination = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910'; //dummy address
    await factorService.withdraw(destination);
    //withdraw x amount
    const amount = 10;
    const destination = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910';
    await factorService.withdraw(destination, 10);





// InvoiceMarketService -> TS_Investor perspective
  //provision - send DAI from msg.sender to some destination
  const amount = 10; // amount of dai to send to an address
  const TSInvestorAddress = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910'; //dummy address
  await marketService.provision(TSInvestorAddress, 10);

  //buy
    //entire invoice
    await FactoredInvoice.buy();
    //50% of invoice
    const share = 0.5
    await FactoredInvoice.buy(share);

  //withdraws DAI from TSAdmin to destination
    //withdraw all
    const destination = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910'; //dummy address
    await marketService.withdraw(destination);

    //withdraw x amount
    const amount = 10;
    const destination = '0x1C00F341bF965E3F04E126de0E78Acb0D0404910';
    await marketService.withdraw(destination, 10);


  TODO:
    CREATE INDEX.JS file in another project that uses the above to run the steps described below:
    
TradeShift App > TradeShift Cash Service
  POST /api/cash/early-payments
  InvoiceID (string)
  PercentageForSale = 100.00
  MaxDiscount = TBD

(TS admin) TradeShift Cash Service > EarlyPaymentsFactory.build
  TokensAmount (invoice face value with 18 decimals, needs investors’ feedback)
  DenominatingCurrency
  MaxDiscount

(TS admin) EarlyPaymentsFactory.build > new TokenContract
  TokensAmount (invoice face value with 18 decimals, needs investors’ feedback)

(TS admin) EarlyPaymentsFactory.build > new SaleContract
  TokenContractAddress
  BeneficiaryAddress (= TS admin)
  DaiAddress
  TokenSaleAmount (in DAI) (not actually needed for the alpha version)
  PricePerToken

(TS admin) EarlyPaymentsFactory.build > new IOUContract
  TokenContractAddress
  DaiAddress
  DenominatingCurrency
  (later: DueDate, …)

(TS investor) > SaleContract.buy

(TS investor) SaleContract.buy > DAI.transferFrom
  From: TS investor
  To: TS admin
  Amount: PricePerToken x SaleContract.TokenSaleAmount

(TS investor) SaleContract.buy > TokenContract.transferFrom
  From: TS admin
  To: TS investor
  Amount: SaleContract.TokenSaleAmount
  (DAI exchange is handled manually)
  (Payment exceptions are handled manually)

(TS admin) > IOUContract.settle()

(TS admin) IOUContract.settle > DaiContract.transferFrom
  From: TS admin
  To: IOUContract
  Amount: TokenContract.totalSupply()

(TS investor) > IOUContract.redeem()

(TS investor) IOUContract.redeem > TokenContract.transferFrom
  From: TS investor
  To: IOUContract
  Amount: TokenContract.totalSupply()

(TS investor) IOUContract.redeem > DaiContract.transferFrom
  From: IOUContract
  To: TS investor
  Amount: DaiContract.balanceOf(this)

(TS investor) IOUContract.redeem > TokenContract.burn()
  Amount: TokenContract.totalSupply()
