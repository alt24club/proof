# Alt24 Club — Daily Crypto‑Prediction Proof Ledger

**Live proof logging began on 2025‑07‑18 (09 → 10 UTC run).**  
Every calendar day, after the model closes its 09:00–10:00 UTC candle, we:

1. **Run the model** and save that day’s full prediction payload (`predictions/YYYY‑MM‑DD.json`) *locally*.  
2. **Commit a SHA‑256 hash** of that JSON to the repo (`hashes/YYYY‑MM‑DD.txt`).  
   No plaintext predictions are published yet.  
3. One day later, **reveal yesterday’s JSON** in a new commit *together with* today’s new hash.
4. Append the day’s **net % return** to `equity_daily/equity_daily.csv`  
   (details below).

Because each hash is committed **before** its corresponding JSON is ever added to the repo, anyone can independently verify we did **not** tamper with past predictions.

---

## Repository Layout
```
/
├── predictions/           # Plain‑text prediction payloads (revealed after 24 h)
│   ├── 2025-07-18.json    # First live day — revealed 2025‑07‑19
│   └── YYYY-MM-DD.json
├── hashes/                # SHA‑256 commitment for each prediction payload
│   ├── 2025-07-18.txt     # Hash for the JSON above (committed on 2025‑07‑18)
│   └── YYYY-MM-DD.txt
├── equity_daily/          # Convenience log of realised day‑by‑day returns
│   └── equity_daily.csv   # Starts at 100 on 2025‑07‑18
├── .gitattributes         # Prevents Git from altering newlines in proof files
└── README.md
```
> **Line‑ending safety:** `.gitattributes` sets `-text` on both folders so Git never rewrites CRLF/LF, ensuring hashes stay valid across OS platforms.

---

## Daily workflow (UTC timeline)

| Phase              | Time (approx) | Public? | Files touched |
|--------------------|---------------|---------|---------------|
| Model run          | 10:00 UTC     | No      | `predictions/YYYY‑MM‑DD.json` (local) |
| Commit hash        | 10:01 UTC     | **Yes** | `hashes/YYYY‑MM‑DD.txt` |
| Reveal + new hash  | ~10:05 UTC next day | **Yes** | prior‑day `predictions/`, new‑day hash |
| Append daily PnL   | ~10:05 UTC next day | **Yes** | `equity_daily/equity_daily.csv` |

All of the above happen in **one atomic commit** each day; branch protection
forbids force‑pushes or merge commits.

---

## How to Verify a Day’s Integrity

Example for **2025‑07‑18** on Windows (PowerShell):

```powershell
git clone https://github.com/alt24club/proof.git
cd proof

# 1) hash committed on 2025‑07‑18
Get-Content .\hashes\2025-07-18.txt

# 2) compute hash of JSON revealed 2025‑07‑19
Get-FileHash .\predictions\2025-07-18.json -Algorithm SHA256
```
---

### macOS / Linux verification

```bash
git clone https://github.com/alt24club/proof.git
cd proof

# 1) hash committed on 2025‑07‑18
cat hashes/2025-07-18.txt

# 2) compute hash of the JSON revealed 2025‑07‑19
# Option A (most Linux distros):
sha256sum predictions/2025-07-18.json

# Option B (macOS built‑in):
shasum -a 256 predictions/2025-07-18.json
```


---

### Quick Verification Snippet (Python 3; cross‑platform)

```python
import hashlib, pathlib

day  = "2025-07-18"
repo = pathlib.Path(__file__).resolve().parent

committed = (repo / f"hashes/{day}.txt").read_text().strip()
actual    = hashlib.sha256((repo / f"predictions/{day}.json").read_bytes()).hexdigest()

print("MATCH ✅" if committed == actual else "MISMATCH ❌")
```

---

## Payload Format
```jsonc
{
  "model_timestamp_utc": "2025-07-18T09:00:00Z",   // candle start
  "trade_executed_utc":  "2025-07-18T10:00:00Z",   // candle close (BUY)
  "threshold": -0.042,
  "predictions": [
    {"coin":"DOGEUSDT","pred_pct":-0.0398,"start_price":0.21606},
    {"coin":"ETHUSDT","pred_pct":-0.0408,"start_price":3466.02}
  ]
}
```
*Prediction rule:* `pred_pct > threshold` ⇒ eligible coin.  
We always trade the **highest** eligible coin out of ADA, AVAX, DOGE, SOL, and XLM at 10 UTC.

---

### `equity_daily/equity_daily.csv`

| Column            | Meaning                                                                    |
|-------------------|----------------------------------------------------------------------------|
| **`date`**        | Entry day (09:00 – 10:00 UTC candle)                                       |
| **`coin`**        | Coin actually traded - using -4.2% as threshold                                                       |
| **`entry_px`**    | Close price of the 09:00 – 10:00 candle (day *D*)                          |
| **`exit_px`**     | Close price of the 09:00 – 10:00 candle (day *D + 1*)                      |
| **`net_ret_pct`** | `(exit / entry − 1) − 0.20 %` fee/slippage                                 |
| **`cum_index`**   | Performance index (starts at **100** on 18‑Jul‑2025)                       |

---

## Data‑Integrity Notes

| Date | Status | Notes |
|------|--------|-------|
| **2025‑07‑18 →** | **Valid** | Official proof start. All subsequent days follow commit → reveal protocol. |

---

## Disclaimer

This repository publishes historical model predictions for **transparency only**.  
Nothing here constitutes financial advice. Trade at your own risk.
