# Rezerve Chain

Consumption-based decentralized AI incentive network. Fork of Bittensor's subtensor.

## What Changed from Bittensor

### Emission Allocation (`pallets/subtensor/src/coinbase/subnet_emissions.rs`)
- **Old (dTAO):** Emissions proportional to subnet token market capitalization
- **New (Rezerve):** `emission = spec_weight * flow_share + cons_weight * (w1 * consumption + w2 * quality)`
- Bootstrap transition: speculative weight decays from 80% to 5% over time

### Epoch Mechanism (`pallets/subtensor/src/epoch/run_epoch.rs`)
- **Old:** Incentive = pure weight-based rank (Yuma consensus)
- **New:** Incentive = blend of weight rank + consumption share, weighted by bootstrap parameter

### New Extrinsic: `report_consumption` (call_index 200)
Validators report verified AI workloads. Parameters:
- `netuid`, `miner_uid`, `compute_units`, `consumer_payment`, `quality_score`, `output_hash`
- Anti-wash-trading: `consumer_payment >= 1.1x * emission_per_subnet`

### New Storage Items
- `SubnetConsumption` — verified consumption per subnet per epoch
- `SubnetQualityScore` — multi-validator quality consensus score
- `MinerConsumption` — per-miner consumption within subnet
- `ConsumerPayments` — total consumer payments per subnet
- `BootstrapSpeculativeWeight` — current speculative emission weight (decays 80% → 5%)
- `ConsumptionWeight` — w1 in emission formula (default 70%)
- `MinConsumerPaymentRatio` — anti-wash-trading ratio (default 1.1x)

### Removed
- Admin-utils pallet functionality is being phased out (trustless from birth)
- Chain renamed from Bittensor to Rezerve in all chain specs

## Building

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install protoc
# macOS: brew install protobuf
# Linux: apt install protobuf-compiler

# Build
cargo build --release -p node-subtensor

# Run dev node
./target/release/node-subtensor --dev

# Run tests
cargo test -p pallet-subtensor --lib
```

## Test Status

957 passed, 0 failed, 7 ignored.
7 new consumption-specific tests added and passing.

## Simulation

Economic simulation (10K epochs, 30% colluders, 20% wash traders) proves:
- Honest miners earn 82% of emissions (PASS)
- Wash trading is unprofitable (PASS)
- Gini 0.127 vs 0.631 (5x more equitable, PASS)
- Honesty is Nash equilibrium for validators (PASS)

See: github.com/rishabhng/rezerve-simulations
