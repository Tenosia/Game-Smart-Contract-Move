# Wolf Game Contracts

Move smart contracts for Wolf Game on Aptos blockchain. This project implements an NFT game where players can mint Sheep and Wolf NFTs, stake them to earn WOOL tokens, and participate in a risky game mechanic.

## Overview

Wolf Game is a play-to-earn NFT game built on Aptos. Players can:
- Mint Sheep and Wolf NFTs with unique traits
- Stake NFTs in the Barn (Sheep) or Pack (Wolves) to earn WOOL tokens
- Participate in risky game mechanics for additional rewards
- Claim WOOL tokens through time-locked pouches

## Project Structure

```
wolfgame-contracts/
├── Woolf/                    # Main contract package
│   ├── Move.toml            # Package configuration
│   └── sources/             # Move source files
│       ├── woolf.move       # Main NFT minting and game logic
│       ├── barn.move        # Staking mechanism for Sheep and Wolves
│       ├── wool.move        # WOOL token implementation
│       ├── wool_pouch.move  # Time-locked WOOL token pouches
│       ├── risky_game.move  # Risky game mechanics
│       ├── traits.move      # NFT trait generation and management
│       ├── config.move      # Configuration management
│       ├── token_helper.move # Token creation utilities
│       ├── random.move      # Random number generation
│       ├── utf8_utils.move  # String utilities
│       └── utils.move       # Test utilities
├── WoolfResourceAccount/    # Resource account setup
├── data/                    # Data fetching scripts
└── cmd.sh                   # Deployment script
```

## Modules

### woolf.move
Main module handling NFT minting, trait generation, and game initialization. Manages the collection and token minting with dynamic pricing based on token index.

### barn.move
Implements the staking system:
- Sheep staking: Earn WOOL tokens at a fixed daily rate
- Wolf staking: Earn WOOL tokens proportional to Alpha rank
- Tax system: Wolves take 20% tax on Sheep claims
- Unstaking mechanics with risk of theft

### wool.move
WOOL token implementation with minting, burning, and transfer capabilities. Used as the in-game currency.

### wool_pouch.move
Time-locked WOOL token pouches that unlock over time. Players can claim unlocked WOOL from their pouches.

### risky_game.move
Risky game mechanics where players can:
- Play it safe: Claim guaranteed WOOL pouches with 20% tax
- Take a risk: Opt into a 50/50 chance to win or lose WOOL
- Claim wolf earnings: Wolves receive taxes and risk game rewards

### traits.move
Manages NFT trait generation using A.J. Walker's Alias Algorithm for weighted random selection. Ensures unique trait combinations and stores trait data.

### config.move
Centralized configuration management using PropertyMap for dynamic settings like mint prices, token limits, and collection metadata.

### token_helper.move
Utilities for creating and managing NFTs, including token data creation, property setting, and collection management.

### random.move
Cryptographically secure random number generation using block height, timestamp, script hash, and sender information.

## Prerequisites

- Aptos CLI installed and configured
- An Aptos account with sufficient funds for deployment
- Move compiler (included with Aptos CLI)

## Setup

1. Clone the repository:
```bash
git clone <repository-url>
cd wolfgame-contracts
```

2. Configure your deployment address:
   - Edit `cmd.sh` and replace `woolf_deployer` with your Aptos account address
   - Update `Woolf/Move.toml` if needed to match your address configuration

3. Fund your account (for devnet):
```bash
aptos account fund-with-faucet --account <your-address>
```

## Deployment

1. Run the deployment script:
```bash
chmod +x cmd.sh
./cmd.sh
```

The script will:
- Fund your account (devnet only)
- Compile the contracts
- Run tests
- Publish the contracts
- Enable minting

Alternatively, deploy manually:

```bash
# Compile
aptos move compile --package-dir Woolf --named-addresses woolf_deployer=<your-address>

# Test
aptos move test --package-dir Woolf --named-addresses woolf_deployer=<your-address>

# Publish
aptos move publish --package-dir Woolf --named-addresses woolf_deployer=<your-address>

# Enable minting
aptos move run --function-id <your-address>::woolf::set_minting_enabled --args bool:true
```

## Usage Examples

### Mint NFTs
```bash
# Mint a single NFT without staking
aptos move run --function-id <address>::woolf::mint --args u64:1 bool:false

# Mint and stake immediately
aptos move run --function-id <address>::woolf::mint --args u64:1 bool:true
```

### Stake NFTs
```bash
# Stake by token index
aptos move run --function-id <address>::barn::add_many_to_barn_and_pack_with_index --args u64:1

# Stake by collection name and token name
aptos move run --function-id <address>::barn::add_many_to_barn_and_pack --args string:"Woolf Game NFT" string:"Sheep #1" u64:1
```

### Claim Earnings
```bash
# Claim from barn/pack by index
aptos move run --function-id <address>::barn::claim_many_from_barn_and_pack_with_index --args u64:1 bool:false

# Claim and unstake
aptos move run --function-id <address>::barn::claim_many_from_barn_and_pack_with_index --args u64:1 bool:true
```

### Risky Game
```bash
# Play it safe
aptos move run --function-id <address>::risky_game::play_it_safe_one --args u64:1 bool:false

# Take a risk
aptos move run --function-id <address>::risky_game::take_a_risk_one --args u64:1

# Execute risk (after opt-in period)
aptos move run --function-id <address>::risky_game::execute_risk_one --args u64:1 bool:false
```

### Claim WOOL Pouches
```bash
# Claim from a single pouch
aptos move run --function-id <address>::wool_pouch::claim --args u64:1

# Claim from multiple pouches
aptos move run --function-id <address>::wool_pouch::claim_many --args vector<u64>[1,2,3]
```

## Configuration

Key configuration values can be set in `config.move`:
- `MINT_PRICE`: Price for paid tokens (0.99 APT)
- `MAX_TOKENS`: Maximum number of tokens (50,000)
- `PAID_TOKENS`: Number of paid tokens (10,000)
- `MAX_SINGLE_MINT`: Maximum tokens per mint (10)
- `TARGET_MAX_TOKENS`: Target maximum tokens (13,809)

Admin functions are available to update configuration:
```bash
aptos move run --function-id <address>::config::set_is_enabled --args bool:true
```

## Recent Optimizations

The codebase has been optimized for better performance and gas efficiency:

- Fixed duplicate error codes
- Optimized mint function to reduce storage reads
- Changed unnecessary mutable borrows to immutable borrows
- Cached repeated function calls in mint_cost
- Removed dead code and commented imports
- Fixed critical bugs in risky_game stage management

## Testing

Run the test suite:
```bash
aptos move test --package-dir Woolf --named-addresses woolf_deployer=<your-address>
```

Tests cover:
- NFT minting
- Staking mechanics
- Claim functionality
- Trait generation
- Token operations

## License

See LICENSE file for details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests to ensure everything works
5. Submit a pull request

## Security

This is experimental software. Use at your own risk. Always audit smart contracts before deploying to mainnet with real funds.

## Support

For issues and questions, please open an issue on the repository.
