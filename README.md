# L2 Native Minting Bot
Our native minting system requires a set of transactions to be executed on each chain to sync with the L1 liquidity pool. All of these calls are permissionless; hence, this bot has an EOA it uses to submit these transactions.

## L2 -> L1 Sync Overview
For all `weETH` minting on L2s, there is corresponding `weETH` minted and deposited into the [EtherFiOFTAdapter](https://etherscan.io/address/0xFE7fe01F8B9A76803aF3750144C2715D9bcf7D0D) contract. The `wETH` from L2s must also be bridged back to mainnet for deposit into the [Liquidity Pool](https://etherscan.io/address/0x308861A430be4cce5502d0A12724771Fc6DaF216).

### fast-sync
If an L2 `SyncPool` contract—[Blast Example SyncPool](https://blastscan.io/address/0x52c4221cb805479954cde5accff8c4dcaf96623b)—has more than 1000 `ETH`, the bot will initiate a `sync` call on said contract. Subsequantly, the `ETH` in the `SyncPool` is sent to be bridged to mainnet via canonical bridge, and a LayerZero message is sent to be executed on mainnet by the [Layer Zero: Executor](https://etherscan.io/address/0xe93685f3bba03016f02bd1828badd6195988d950).
The LayerZero message causes the following actions on Mainnet:
- `Dummy ETH` is deposited into EtherFi through the [Vampire Contract](https://etherscan.io/address/0x9ffdf407cde9a93c47611799da23924af3ef764f).
- `weETH` is minted and deposited in the [EtherFiOFTAdapter](https://etherscan.io/address/0xFE7fe01F8B9A76803aF3750144C2715D9bcf7D0D).

### slow-sync
After `fast-sync` sends ETH to the L2 canonical bridge and the sequencer submits the transaction's state root to mainnet, the bot performs these steps to withdraw the ETH:
- Submit a proof of withdrawal transaction.
- Wait 7 to 14 days (variable for each L2) for the challenge period to pass.
- Submit a relay transaction.

For an optimisitic rollup like an OP stack chain the transaction lifecycle consists these 5 states: <br>
Waiting for state root -> Ready to prove -> In challenge period -> Ready for relay -> Relayed

## Repo Structure
```
project-root/
│
├── src/                   # Source files
│   ├── index.ts           # Entry point for the bot logic and contains the `fastsync` logic
│   │
│   ├── chains/            # Chain-specific logic
│   │   ├── index.ts       # Entry point for slowsync for all chains
│   │   ├── chainConfig.ts # Configurations for each chain
|   |   ├── helpers.ts     # stores the OP stack helper functions for `slowsync`
│   │   └── slowsync/      # holds a file with the `slowsync` logic for each chain 
│   │       ├── Blast.ts   
│   │       ├── Mode.ts      
│   │       └── ...
│   └── abis/              # All contract ABIS
│       ├── Contract1.json
│       ├── Contract2.json
│       └── ...
└── dist/                  # Compiled JavaScript files
```

## Running the Project

1. **Install Dependencies**: Install the project dependencies using:
    ```sh
    pnpm install
    ```

2. **Set Environment Variables**: add the `PRIVATE_KEY` for the EOA assiocated with the bot and the `ALCHEMY_KEY` to an `.env` file. The values are stored in GitHub Secrets for GitHub Actions to access.

4. **Compile and Run**: Compile the TypeScript code and run the bot using:
    ```sh
    pnpm start
    ```

## GitHub Actions

The project is configured to run a daily job at at 7:00 AM, 1:00 PM, and 7:00 PM UTC using GitHub Actions. The GitHub Actions workflow is defined in [main.yml](https://github.com/GadzeFinance/native-minting-bot/blob/main/.github/workflows/main.yml).
