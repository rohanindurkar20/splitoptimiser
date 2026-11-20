# Split Optimizer benchmark: 18 Nov 2026

3,429 real-size sell orders on Ethereum mainnet, quoted by Split Optimizer and by three DEX aggregators
(each with more than $100 billion in all-time volume on DefiLlama, as of 20 Nov 2026) on the same block and
the same on-chain liquidity. The aggregators are anonymized as A, B and C.

| Aggregator | Orders | Split Optimizer ahead | Behind | Median | Size-weighted | Extra output |
|---|---|---|---|---|---|---|
| A | 3,303 | 2,075 | 169 | +0.18 bps | +4.82 bps | $392,560 |
| B | 3,289 | 1,838 | 249 | +0.09 bps | +1.03 bps | $84,060 |
| C (orders of 100+ ETH only) | 656 | 41 | 395 | -0.10 bps | +0.23 bps | $13,932 |

After gas, at each block's base fee. Positive means Split Optimizer's route gives the user more. Before-gas
figures and size buckets are in `summary.json`.

## Browse the results

Open `index.html` (served by GitHub Pages) for the interactive version: every order, searchable, with links to its
source trade, block and every pool in the route.

## Run both, keep the better quote

| Aggregator | Gain after gas, bps of the whole volume | Extra output |
|---|---|---|
| A | +5.3 bps | $433k |
| B | +1.8 bps | $150k |
| C (orders of 100+ ETH only) | +1.1 bps | $67k |

The optimisation is heuristic, so on some orders it returns less than the aggregator (the "Behind" column above).
It is meant to run in parallel with an existing router, keeping the better quote per order, so those orders lose
nothing. The heuristics will be tuned on more order data and improved in later versions.

## Speed

Median time per quote is about 120 ms (p90 about 150 ms), measured on a single 16-vCPU Google Cloud VM (c4d-highcpu-16). This is the first version and it will keep improving: latency can be reduced further with more cores and tuning, and throughput can be raised as needed by running more instances, since each one is stateless.

## Method

- **Orders:** pair and size from executed CoW Swap trades (`order_tx` is that trade's transaction).
- **Same block:** each aggregator's public quote API was called live on 18 Nov 2026. Split Optimizer quoted the same
  order on the on-chain state that aggregator priced (`agg_*_quote_block`), from a block-by-block market recording.
- **Same liquidity:** aggregators limited to the on-chain venue types Split Optimizer covers (Uniswap V2/V3/V4,
  SushiSwap, PancakeSwap, Curve, Balancer V2/V3, Fluid, Ekubo, ERC-4626); RFQ and private market makers excluded.
- **Fees and gas:** aggregator fees added back. After-gas figures charge each route its own gas estimate at the
  block's base fee, priced in the output token.
- **Left out of the totals:** comparisons 1,000 bps or more apart, orders under $1,000 and orders with no route.
  Most are tokens the router does not yet cover well (Aave aTokens, USDY, PEPE, ENA and a few others), where the
  aggregator's quote is far better; run in parallel, those orders keep the aggregator's quote. All of them are
  still in `results.csv`; the counts are in `summary.json` under `excluded`.

## Files

| File | What it holds |
|---|---|
| `results.csv` | One row per order: block, pair, size, Split Optimizer's output and gas, each aggregator's output, gas, quote block and difference |
| `routes.jsonl.gz` | Split Optimizer's route per order: every pool with its amount out |
| `blocks.json` | Base fee of every quoted block |
| `summary.json` | The tables above, by order size, before and after gas |
| `MANIFEST.json` | SHA-256 of every file here and of each aggregator's raw API responses |

SHA-256 of `MANIFEST.json`: `7bbd402d13bdb43a30b478ca0a97a5f59a9598aa633689f8c31593e245f08f2c`

Each aggregator's raw responses (with request ids) are not public, so that it stays unnamed. Each aggregator can
receive its own file and check it against its logs and against the hash in `MANIFEST.json`.

## Contact

Rohan Indurkar · rohanindurkar92@gmail.com · Telegram @mrr0b0t2004
