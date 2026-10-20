# Bonding Curve IDL

IDL for the `lastbuy_program` bonding curve program - a Solana program where tokens are bought/sold along a bonding curve, the **last buyer** automatically becomes the **winner** of a SOL prize pool, and liquidity graduates to a **Raydium CPMM** pool.

Source: [`idl.json`](./idl.json)

## Program Address

| Cluster  | Address | Status |
|----------|---------|--------|
| Mainnet  | `Lbuyii1PVUvt9xToGuyUf6CQCDbdRRzzo8SsHuNYMkf` | **Not deployed yet** |
| Devnet   | `Lbuyii1PVUvt9xToGuyUf6CQCDbdRRzzo8SsHuNYMkf` | Deployed |
| Testnet  | `Lbuyii1PVUvt9xToGuyUf6CQCDbdRRzzo8SsHuNYMkf` | Deployed |

> The same program address is used on all clusters. The program is **not deployed to mainnet yet**.

## Instructions

| Instruction | Description |
|-------------|-------------|
| `initialize_global` | Admin: initialize the global config. |
| `update_global` | Admin: update the global config. |
| `launch` | Launch a new token with its bonding curve and prize pool. |
| `buy_exact` | Buy tokens with a fixed SOL input (`sol_in`, `min_tokens_out`). Tracks the last buyer and defines the winner. |
| `sell_exact` | Sell tokens for SOL (`tokens_in`, `min_sol_out`). |
| `fund_migration` | Fund Raydium pool creation costs. The funder earns CPMM trading fees. |
| `prepare_migration` | Permissionless: withdraw SOL liquidity to the pool creator; splits an unclaimed prize among fee recipients & creator. Locks `claim_creator_fee`. |
| `migrate` | Permissionless: withdraw token liquidity, create & seed the Raydium CPMM pool, burn LP. Unlocks `claim_creator_fee`. |
| `claim_prize_idempotent` | Winner claims the SOL prize pool. Fails if called while the winner is undefined. |
| `define_winner_idempotent` | Manually define the winner; recommended to pair with a prize claim. |
| `claim_creator_fee` | Claim accumulated Raydium CPMM trading fees (bonding-curve fees are paid out automatically). |
| `emergency_drain_curve` | Admin: drain all curve liquidity to a recipient. |

## Accounts

| Account | Description |
|---------|-------------|
| `CurveState` | Per-mint curve: reserves, status, last buyer, migration funder, creator. |
| `GlobalConfig` | Protocol-wide fees, protocol config, Raydium config, admin toggles. |
| `PrizePool` | SOL prize pool per curve: winner, alternative winner, claimed flag. |
| `PoolCreatorState` | Tracks the Raydium pool creator PDA and fee-claim lock. |

`CurveStatus`: `Active` => `Graduated` / `Migrating` => `Migrated`, plus `Inactive`, `WinnerDefined`, `AlternativeWinnerDefined`.

## PDA Seeds

| Seed | PDA |
|------|-----|
| `"curve"` + `mint` | `CurveState` |
| `"global_config"` | `GlobalConfig` |
| `"prize_pool"` + `curve` | `PrizePool` |
| `"mint_authority"` | Mint authority |
| `"pool_state"` + `curve` | Raydium pool state |
| `"pool_creator"` + `migration_funder` + `curve` | Pool creator |
| `"pool_creator_state"` + `pool_creator` | Pool creator state |
| `"nft_mint"` + `curve` | Raydium NFT mint |

## Key Constants

- `MIN_SOL_IN_THRESHOLD` = `1_000_000` lamports (0.001 SOL) - buys below this don't update the last buyer (anti-dust-grift).
- Trades use **Token-2022** (`TokenzQd...`) for the bonding curve; wSOL/legacy SPL Token is used on the Raydium side.

## Events

`LaunchEvent`, `BuyEvent`, `SellEvent`, `GraduationEvent`, `DefineWinnerEvent`, `DefineAlternativeWinnerEvent`, `ClaimPrizeEvent`, `ClaimCreatorFeeEvent`, `FundMigrationEvent`, `PrepareMigrationEvent`, `MigrationEvent`, `EmergencyDrainCurveEvent`

## Errors

| Code | Name | Message |
|------|------|---------|
| 6000 | `SlippageExceeded` | Slippage exceeded |
| 6001 | `MigrationThresholdExceeded` | Migration threshold exceeded |
| 6002 | `MigrationFunded` | Migration already funded |
| 6003 | `MigrationUnfunded` | Migration unfunded |
| 6004 | `MigrationActive` | Migration is still active |
| 6005 | `InsufficientOutputAmount` | Insufficient output amount |
| 6006 | `InsufficientCurveBalance` | Insufficient curve balance for sell |
| 6007 | `ZeroTradeAmount` | Trade amount must be greater than zero |
| 6008 | `MetadataTooLong` | Token metadata exceeds maximum length |
| 6009 | `FeeExceededMaxAllowed` | Fee exceeded maximum allowed |
| 6010 | `CanonicalWinnerExists` | Canonical winner exists |
| 6011 | `CurveActive` | Bonding curve is still active |
| 6012 | `CurveClosed` | Bonding curve is graduated or migrated |
| 6013 | `InvalidFeeRecipient` | Invalid fee recipient |
| 6014 | `InvalidCreator` | Invalid creator |
| 6015 | `WinnerNotDefined` | Winner not defined |
| 6016 | `InvalidMintOrder` | Invalid mint order |
| 6017 | `InstructionDisabled` | Instruction disabled |
