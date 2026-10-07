# Stellar Trade Contracts

This repository contains the Soroban contracts for [Stellar Trade](https://github.com/Stellar-tradehub/stellar-trade), an open-source prediction-market prototype on Stellar. The contracts define how XLM enters a market pool, how positions can be reduced, and how resolved or cancelled markets account for payouts and refunds.


## Workspace map

| Crate | Responsibility |
| --- | --- |
| `prediction_market` | Market creation, Buy positions, position reductions, resolver actions, proportional claims, cancellation refunds, fee accounting, disputes, and governance. |
| `stellar_trade_token` | Reward token with admin controls, authorized minters, pause controls, and an optional supply cap. Fresh deployments use name `Stellar Trade` and symbol `STRD`. |
| `referral_registry` | Referral registration, bounded referral chains, and referral-fee distribution. |
| `leaderboard` | Participant points and win/loss records, score decay, and authorized reward minting. |
| `cross_contract_invariants` | Integration tests for accounting and behavior across the market, referral, leaderboard, and token contracts. |

## Market lifecycle

1. An authorized creator publishes a market question, category, and close time.
2. A participant calls `place_bet` to stake XLM on YES or NO. The contract credits the selected side after applying configured fees.
3. Before a market closes, the participant may call `reduce_position` to reduce their existing stake. The contract chooses the participant's recorded side and calculates the returned amount from the reduced stake and retained fee rules. This is not a peer-to-peer sale, an order book, or a short position.
4. An authorized resolver submits the outcome. The market contract calculates proportional payouts for winning positions; participants claim their payout through the contract. Cancelled markets use a separate refund path.

Resolution is permissioned. There is no automated oracle in this workspace. External price or network data does not settle a market unless an authorized resolver uses it and submits the result. Resolver selection, criteria, evidence, and dispute handling therefore remain material trust assumptions.

## Accounting and safety properties

The [invariant matrix](INVARIANT_MATRIX.md) lists the key rules and the tests that protect them. Among these are fee conservation, market-level fee isolation, proportional payout bounds, cancellation behavior, referral accounting, reward supply tracking, storage TTL handling, and emergency pause paths. Review the matrix before changing fees, rewards, payout formulas, or storage lifetimes. Add or update regression tests with any behavior change.

The `issues/` directory contains historical security and correctness reports from prior review work. Read [`issues/README.md`](issues/README.md) for their context; these reports document known design risks and should not be read as proof that the contracts have been audited or that every issue remains current.

## Build and test

The workspace pins Rust 1.91.0 and the `wasm32v1-none` target in `rust-toolchain.toml`. Install the toolchain and target, then run from this repository's root:

```bash
cargo test --workspace --all-features
cargo fmt --all -- --check
cargo clippy --workspace --all-features -- -D warnings
cargo build --release --target wasm32v1-none -p prediction_market
```

The GitHub Actions workflow runs workspace checks, tests, WASM builds, and cross-contract invariant tests on changes to the default branch and pull requests. For local frontend development and deployment instructions, see the [application README](https://github.com/Stellar-tradehub/stellar-trade/blob/main/README.md).

## Deployment and secrets

The deployed Testnet contracts retain their historical IDs and token metadata. A fresh deployment creates new addresses and initializes STRD; it does not migrate existing holders or balances. Confirm network, contract IDs, admin identity, fee settings, and resolver configuration before any deployment. Never commit secret keys, wallet recovery phrases, `.deploy.env`, deployment output, or local identity files.

## License

The contract source is licensed under the MIT License. See [`LICENSE`](LICENSE). Third-party dependencies retain their own licenses.
