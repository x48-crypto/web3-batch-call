# web3-batch-call

web3-batch-call is a tool for querying large amounts of contract data in one json-rpc call.

## Interact

https://batchcall.finance

## Features

- Call out to many different contracts in a single batched request
- Call a large number of methods on various contracts
- Supply custom arguments to methods
- Automatically fetch and populate state for viewable no-input methods
- Supply your own ABIs or automatically fetch and cache ABIs
- Supports custom contract namespaces
- Supports historical blocks
- Isomorphic (works on node and in the browser)

## Use Cases

- Read the complete state of all vaults/staking pools for a protocol as well as user balances
- Track bulk contract state changes when a contract is touched within a block (give drizzle superpowers)
- Support real-time contract state tracking cloud infrastructure
  - Users can subscribe to updates from specific contracts with their own custom arguments (delivered via pub-sub/websocket)
- Find all uniswap token pairs
  - Batch read uniswap token factory to obtain pair addresses
- Find all uniswap token prices
  - Pass an array of uniswap token addresses with price arguments
- Contract analytics tooling

## Request Schema

```
/**
 * Array of contract request configurations
 */
[
  {
    namespace, // Specify a namespace to identify this configuration. Namespace will be used to group contract results
    addresses, // Specify a list of addresses to iterate through for this contract config
    abi, // Specify an ABI to use for all addresses in this contract config. If no ABI is specified a unqiue ABI will be fetched and cached for every address. Specifying an ABI is recommended because fetching ABIs via etherscan is time consuming
    allMethods, // Specify true/flase. If true the contract ABI will be used to fetch state for all viewable methods with no inputs
    methods: [
      // Array of methods with custom arguments
      {
        name, // method name
        args, // array of method arguments
      },
    ],
  },
];
```

## Installation

```
npm install --save web3-batch-call
```

## Configuration

```
import BatchCall from "web3-batch-call";

const provider = "https://mainnet.infura.io/v3/<your_infura_api_key>";
const etherscanApiKey = "<your_etherscan_api_key>"

const options = {
  provider, // Only required if not providing your own web3 instance
  web3, // Only required if not providing your own provider
  etherscan: {
    apiKey: etherscanApiKey, // Only required if not providing abi in contract request configuration
    delayTime: 300, // delay time between etherscan ABI reqests. default is 300 ms
  },
}

const batchCall = new BatchCall(options);
const contracts = yourContractsArray
const result = await batchCall.execute(contracts, blockNumber);

console.log(result);
```

### Example Request

```
import BatchCall from "web3-batch-call";

const contracts = [
  {
    namespace: "vaults",
    addresses: [
      "0x5dbcF33D8c2E976c6b560249878e6F1491Bca25c",
      "0x29E240CFD7946BA20895a7a02eDb25C210f9f324",
    ],
    allMethods: true,
    methods: [
      {
        name: "balanceOf",
        args: ["0x1A91C0Df156A5F38aEC0813d055aA0184Fc47826"],
      },
      {
        name: "balanceOf",
        args: ["0xF98304d7FB1EE427A8a45B2040677cE57eE454dA"],
      },
    ],
  },
  {
    namespace: "stakingPools",
    addresses: ["0xBa37B002AbaFDd8E89a1995dA52740bbC013D992"],
    allMethods: false,
    methods: [
      {
        name: "balanceOf",
        args: ["0x29E240CFD7946BA20895a7a02eDb25C210f9f324"],
      },
    ],
  },
];

const provider = "https://mainnet.infura.io/v3/<your_infura_api_key>";
const etherscanApiKey = "<your_etherscan_api_key>";
const batchCall = new BatchCall({
  provider,
  etherscan: {
    apiKey: etherscanApiKey,
    delay: 300,
  },
});

const result = await batchCall.execute(contracts);
console.log(result);
```

### Example Response

```
{
  vaults: [
    {
      address: '0x5dbcF33D8c2E976c6b560249878e6F1491Bca25c',
      balanceOf: [
        {
          value: '163407599086590',
          input: '0x70a082310000000000000000000000001a91c0df156a5f38aec0813d055aa0184fc47826',
          args: [ '0x1A91C0Df156A5F38aEC0813d055aA0184Fc47826' ]
        },
        {
          value: '0',
          input: '0x70a08231000000000000000000000000f98304d7fb1ee427a8a45b2040677ce57ee454da',
          args: [ '0xF98304d7FB1EE427A8a45B2040677cE57eE454dA' ]
        }
      ],
      available: [ { value: '175106794577135070165115' } ],
      balance: [ { value: '69152742956129694188230438' } ],
      controller: [ { value: '0x9E65Ad11b299CA0Abefc2799dDB6314Ef2d91080' } ],
      decimals: [ { value: '18' } ],
      getPricePerFullShare: [ { value: '1115942680749254819' } ],
      governance: [ { value: '0xFEB4acf3df3cDEA7399794D0869ef76A6EfAff52' } ],
      max: [ { value: '10000' } ],
      min: [ { value: '10000' } ],
      name: [ { value: 'yearn Curve.fi yDAI/yUSDC/yUSDT/yTUSD' } ],
      symbol: [ { value: 'yyDAI+yUSDC+yUSDT+yTUSD' } ],
      token: [ { value: '0xdF5e0e81Dff6FAF3A7e52BA697820c5e32D806A8' } ],
      totalSupply: [ { value: '61968006196966914927951406' } ]
    },
    {
      address: '0x29E240CFD7946BA20895a7a02eDb25C210f9f324',
      balanceOf: [
        {
          value: '0',
          input: '0x70a082310000000000000000000000001a91c0df156a5f38aec0813d055aa0184fc47826',
          args: [ '0x1A91C0Df156A5F38aEC0813d055aA0184Fc47826' ]
        },
        {
          value: '344733454224',
          input: '0x70a08231000000000000000000000000f98304d7fb1ee427a8a45b2040677ce57ee454da',
          args: [ '0xF98304d7FB1EE427A8a45B2040677cE57eE454dA' ]
        }
      ],
      aave: [ { value: '0x24a42fD28C976A61Df5D00D0599C34c4f90748c8' } ],
      availableToBorrowETH: [ { value: '12871798970841184666681' } ],
      availableToBorrowReserve: [ { value: '6918769017550' } ],
      balance: [ { value: '2991115423000586249520147' } ],
      controller: [ { value: '0x2be5D998C95DE70D9A38b3d78e49751F10F9E88b' } ],
      credit: [ { value: '6571575672124' } ],
      debt: [ { value: '6568313166454' } ],
      decimals: [ { value: '18' } ],
      getAave: [ { value: '0x398eC7346DcD622eDc5ae82352F02bE94C62d119' } ],
      getAaveCore: [ { value: '0x3dfd23A6c5E8BbcFc9581d2E864a68feb6a076d3' } ],
      getAaveOracle: [ { value: '0x76B47460d7F7c5222cFb6b6A75615ab10895DDe4' } ],
      getPricePerFullShare: [ { value: '1050253924653889449' } ],
      getReservePrice: [ { value: '1860417501753607' } ],
      getUnderlyingPrice: [ { value: '27108074719526996' } ],
      governance: [ { value: '0x2D407dDb06311396fE14D4b49da5F0471447d45C' } ],
      healthFactor: [ { value: '2' } ],
      insurance: [ { value: '6823725788804860163571' } ],
      locked: [ { value: '1000496703733107980' } ],
      ltv: [ { value: '65' } ],
      max: [ { value: '100' } ],
      maxSafeETH: [
        {
          value: Result {
            '0': '26412214466011430693228',
            '1': '12219804771869674491885',
            '2': '40604624160153186894572',
            maxBorrowsETH: '26412214466011430693228',
            totalBorrowsETH: '12219804771869674491885',
            availableBorrowsETH: '40604624160153186894572'
          }
        }
      ],
      name: [ { value: 'yearn Aave Interest bearing LINK' } ],
      over: [ { value: '0' } ],
      reserve: [ { value: '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48' } ],
      shouldBorrow: [ { value: true } ],
      shouldRebalance: [ { value: false } ],
      symbol: [ { value: 'yaLINK' } ],
      token: [ { value: '0xA64BD6C70Cb9051F6A9ba1F163Fdc07E0DfB5F84' } ],
      totalSupply: [ { value: '2847992616629646531247502' } ],
      underlying: [ { value: '0x514910771AF9Ca656af840dff83E8264EcF986CA' } ]
    }
  ],
  stakingPools: [
    {
      address: '0xBa37B002AbaFDd8E89a1995dA52740bbC013D992',
      balanceOf: [
        {
          value: '739758310196',
          input: '0x70a082310000000000000000000000001a91c0df156a5f38aec0813d055aa0184fc47826',
          args: [ '0x1A91C0Df156A5F38aEC0813d055aA0184Fc47826' ]
        }
      ]
    }
  ]
}
```

## How it works

Under the hood the web3 [BatchRequest](https://web3js.readthedocs.io/en/v1.3.0/include_package-core.html?highlight=batchrequest#batchrequest) class is utilized.

```javascript=
var contract = new web3.eth.Contract(abi, address);

var batch = new web3.BatchRequest();
batch.add(web3.eth.getBalance.request('0x0000000000000000000000000000000000000000', 'latest', callback));
batch.add(contract.methods.balance(address).call.request({from: '0x0000000000000000000000000000000000000000'}, callback2));
batch.execute();
```

`web3-batch-call` essentially takes a user-generated contract configuration file and programatically builds and executes a batched json-rpc request using the provided configuration. The results are grouped by namespace and formatted into a JSON object that can be consumed by your application.
