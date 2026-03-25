# FundTrace IQ v4.0 — Full-Stack Fraud Detection Engine

**Team Nextgen · AI-Thon 1.0 · Domain: Cybersecurity & Threat Detection**

---

## Architecture

```
Browser (HTML/JS)              Python Backend (Flask)
──────────────────             ──────────────────────────────
Canvas graph viz    ←──API──→  fraud_engine.py  (detection)
Alert rendering                 ├─ detect_circular()    (DFS)
Timeline / detail               ├─ detect_structuring() (sliding window)
Config modal                    ├─ detect_dormant()     (inactivity+sweep)
                                ├─ detect_profile()     (category deviation)
                                ├─ detect_mule()        (retention scoring)
                                ├─ score_accounts()     (weighted scoring)
                                └─ networkx betweenness centrality

                               app.py  (Flask REST API)
                               └─ /api/analyze, /api/inject,
                                  /api/config, /api/str, /api/reset …
```

## Quick Start

```bash
pip install -r requirements.txt
python app.py
# open http://localhost:5000
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/analyze` | Run full detection pipeline, return all results |
| GET | `/api/accounts` | All accounts with computed scores |
| GET | `/api/transactions` | All transactions with flag labels |
| GET | `/api/alerts` | Sorted alert list (crit → high → med) |
| GET | `/api/patterns` | Raw detected patterns (circular, structuring…) |
| GET | `/api/graph` | Node/edge list for graph rendering |
| GET | `/api/config` | Current detection thresholds |
| POST | `/api/config` | Update thresholds + re-analyze |
| POST | `/api/inject` | Inject new transaction + re-analyze |
| GET | `/api/str` | Generate FIU-IND Suspicious Transaction Report |
| POST | `/api/reset` | Reset to baseline data |
| GET | `/api/profiles` | Account profile category definitions |

## Detection Algorithms

### 1. Circular Transactions
DFS from every node; confirms cycle when origin is revisited.
Filters: return ratio ≥ 80%, time span ≤ 72h, entry ≥ ₹5L.

### 2. Structuring (Smurfing)
Sliding 24h window per account; flags ≥3 sub-₹1L txns with aggregate ≥₹2.5L.

### 3. Dormant Account Misuse
Account inactive ≥180 days, receives ≥₹5L spike, forwards ≥80% within 24h.

### 4. Profile–Behaviour Mismatch
Observed monthly volume vs declared category ceiling (e.g. salaried ₹1.5L/month).
Flags ≥3× deviation; critical at ≥10×.

### 5. Mule Account Detection
Retention score = (in − out) / in. Flags < 10%; critical < 5%.
Chains detected via DFS through consecutive mule nodes.
NetworkX betweenness centrality augments hub scoring.

## Scoring Model

| Component | Max pts |
|-----------|---------|
| Circular | 35 |
| Mule | 30 |
| Structuring | 20 |
| Dormant | 20 |
| Profile mismatch | 18 |
| Network hub | 10 |
| Velocity | 8 |

Risk tiers: Low < 20 · Medium 20–44 · High 45–69 · Critical ≥ 70

## Tech Stack
- **Backend**: Python 3, Flask, Flask-CORS, NetworkX
- **Frontend**: Vanilla JS, HTML5 Canvas (force-directed graph)
- **Report**: Dynamic STR generation for FIU-IND filing
