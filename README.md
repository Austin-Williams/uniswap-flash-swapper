# uniswap-flash-swapper
An abstraction for UniswapV2 flash swaps.
Enables the user to borrow any token and repay using any other token.
Abstracts away most of the nitty-gritty details of the UniswapV2 core contracts.

## Warning
These contracts have not been audited. Be careful.

## How to use
1. Inherit the `UniswapFlashSwapper` contract into your contract.
2. Override the `execute` function to do whatever you want.
3. Call the `startSwap` function, telling it:
  - The address of the token you want to borrow (`_tokenBorrow`) -- use the zero address for ETH
  - How much of that token you want (`_amount`)
  - The address of the token you want to use to pay back the loan (`_tokenPay`) -- use the zero address for ETH
  - Any custom `_userData` you want to be made available to you in the `execute` function

**Note:** If you want to do a traditional flash loan, where you pay back the loan using the same token that you borrowed, then just enter the address of the token you want to borrow for both the `_tokenBorrow` and `_tokenPay` paramters (using the zero address if you want ETH).

### Example 1

```
pragma solidity 0.5.17;

import './UniswapFlashSwapper.sol';

contract ExampleContract is UniswapFlashSwapper {

    // @notice Call this function to make a flash swap
    function flashSwap(address _tokenBorrow, uint256 _amount, address _tokenPay, bytes calldata _userData) external {

        // Start the flash swap
        // This will borrow _amount of the requested _tokenBorrow token for this contract and then
        // run the `execute` function below
        startSwap(_tokenBorrow, _amount, _tokenPay, _userData);

    }

    // @notice This is where your custom logic goes
    // @dev When this code executes, this contract will hold _amount of _tokenBorrow
    function execute(address _tokenBorrow, uint _amount, address _tokenPay, uint _amountToRepay, bytes memory _userData) internal {
        // do whatever you want here
        // <insert arbitrage, liquidation, CDP collateral swap, etc>
        // be sure this contract is holding at least _amountToRepay of the _tokenPay tokens before this function finishes executing
        // DO NOT pay back the flash loan in this function -- that will be handled for you automatically
    }

}
```

### Example 2

See the [`Example.sol` contract](https://github.com/Austin-Williams/uniswapv2-flash-loan-template/blob/master/Example.sol) for a more complete example that should work on mainnet.

## Fees

Each UniswapV2 pair charges a `0.3%` fee.

- If you are doing a traditional "flash loan", where you repay using the same token that you borrowed, you'll be charged a `0.3%` fee.
- If you are borrowing ETH or WETH and repaying with a non-{ETH, WETH} token, you'll be charged a `0.3%` fee.
- If you are borrowing a non-{ETH, WETH} token and repaying with ETH or WETH, then you'll be charged a `0.3%` fee.
- If you are swapping a non-{ETH, WETH} token for another non-{ETH, WETH} token, then you'll be charged a `0.6%` fee because your swap will touch _two_ UniswapV2 pairs (they are routed through the WETH).

## Testnet

If you want to test this on Rinkeby instead of mainnet, you'll need to use the WETH and DAI addresses for Rinkeby. See corresponding PR #4 for help.

## Testing

1) `$ yarn install`
2) Rename .env.sample to .env
3) Populate all variables in the .env file.
  - `NETWORK`: The network that will be forked in the tests.
  - `SIGNER_ADDRESS_<NETWORK>`: The address that will be used to sign transactions in the forked network.
  - `CHAIN_PROVIDER_<NETWORK>`: E.g. your Infura endpoint.
4) Run `yarn start-chain`
5) Run `yarn compile`
6) Run `yarn test`
