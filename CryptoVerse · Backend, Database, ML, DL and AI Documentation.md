# CryptoVerse · Backend, Database, ML, DL and AI Documentation

## 1. Overview

Ei document-e CryptoVerse-er backend, database, machine learning, deep learning ar AI agent-er design deya ache. Frontend React app ei backend-er API use korbe.

**Core idea:** Prottek 4 ghonta-y system notun price, news ar social data collect kore, sentiment score kore, model update kore, forecast + uncertainty banay, ar frontend-e pathay.

**Research contribution (proposed, confirm korte hobe):**

1. Multi-crypto forecasting, notun coin shoho (transfer learning)
2. Context-aware, source-weighted sentiment (fake/spam filter shoho)
3. 4-hour intraday cycle ar news-price lead-lag
4. Realistic evaluation: fee, slippage, prediction uncertainty

**Out of scope:** Reinforcement Learning trading, leverage/futures, real money trading. App paper-trading only.

## 2. Tech Stack

| Layer | Tool |
| --- | --- |
| Language | Python 3.11+ |
| API | FastAPI + Uvicorn |
| Database | PostgreSQL + TimescaleDB extension |
| Cache / queue | Redis |
| Scheduler | APScheduler (shuru), Celery (pore dorkar hole) |
| Data access | SQLAlchemy 2 + Alembic (migration) |
| ML | scikit-learn, XGBoost, pandas, numpy |
| DL | PyTorch |
| NLP | Hugging Face Transformers (FinBERT, multilingual model) |
| Exchange data | CCXT |
| Backtest | vectorbt (ba nijer simple backtester) |
| Experiment tracking | MLflow |
| Validation | Pydantic |
| Container | Docker + docker-compose |
| Testing | pytest |

## 3. System Architecture

```
[Exchange APIs] [News feeds] [Social APIs]
        \             |            /
         v            v           v
      Ingestion workers (every 4h + live price stream)
                      |
                      v
   PostgreSQL / TimescaleDB  <---->  Redis (cache, pub/sub)
                      |
         Feature pipeline (price + sentiment + macro)
                      |
         Model training / inference (XGBoost, GRU/LSTM, Transformer)
                      |
         Forecast + uncertainty + backtest metrics
                      |
                      v
               FastAPI (REST + WebSocket)
                      |
                      v
               React frontend
```

## 4. Repository Structure

```
cryptoverse-backend/
  app/
    api/           routers: auth, markets, forecast, sentiment, orders, wallet, community, agent
    core/          config, security, logging
    db/            models, session, migrations
    schemas/       Pydantic request/response models
    services/      business logic (paper trading, wallet, community)
  ingestion/       price.py, news.py, social.py, macro.py
  nlp/             sentiment model, source weighting, spam filter
  ml/
    features/      feature engineering
    models/        xgb.py, gru.py, lstm.py, transformer.py
    training/      walk-forward training, transfer learning
    inference/     predict + uncertainty
    evaluation/    metrics, backtest
  agent/           tools, prompts, orchestration
  scheduler/       4-hour jobs
  tests/
  docker/
  docker-compose.yml
```

## 5. Database Design

**Standard tables (PostgreSQL):**

| Table | Main columns |
| --- | --- |
| `users` | id, email, password\_hash, name, created\_at |
| `assets` | id, symbol, name, listed\_at, is\_new |
| `wallets` | id, user\_id, account (spot/funding/earn), asset\_id, free, locked |
| `orders` | id, user\_id, asset\_id, side, type, price, amount, stop, status, created\_at |
| `trades` | id, order\_id, price, amount, fee, filled\_at |
| `posts` | id, user\_id, type, title, body, asset\_id, direction, media\_url, created\_at |
| `votes`, `ratings`, `comments` | Community interaction |
| `alerts` | id, user\_id, asset\_id, threshold, active |
| `sources` | id, platform, handle, category, weight, verified, spam\_score |

**Time-series tables (TimescaleDB hypertable):**

| Table | Main columns |
| --- | --- |
| `prices` | ts, asset\_id, open, high, low, close, volume |
| `sentiment_items` | ts, source\_id, asset\_id, text\_hash, raw\_score, weighted\_score, relevance, is\_flagged |
| `sentiment_agg` | ts (4h bucket), asset\_id, score, shift, mention\_count |
| `macro` | ts, indicator, value |
| `forecasts` | ts, asset\_id, horizon, model\_version, point, lower, upper, confidence |
| `model_runs` | id, asset\_id, model, version, train\_end, metrics (JSON), created\_at |

**Rules:**

- Passwords hash kora hobe (argon2 ba bcrypt). Plain text kokhono na.
- Time-series data Timescale hypertable-e, 4h bucket-er jonno continuous aggregate.
- Raw social text-er shob copy rakhar dorkar nei, text hash + score rakhle cholbe (storage ar platform terms duita-er jonno bhalo).
- Schema change shudhu Alembic migration diye.

## 6. Data Ingestion

| Source | Method | Frequency |
| --- | --- | --- |
| Price (OHLCV) | CCXT, exchange REST/WebSocket | Live stream + 4h candle close |
| News | Licensed news API / RSS feed | Every 4h |
| Social (X, Reddit, YouTube, Facebook, LinkedIn, Telegram, Instagram) | Official APIs only | Every 4h |
| Macro | FRED or similar public API | Daily |

**Honest warnings:**

- X, Facebook, Instagram API access shimito ar costly hote pare. Scraping platform terms bhange. Tai prothome Reddit, YouTube, RSS news, Telegram public channel diye shuru kore, baki platform pore add kora bhalo.
- "Sob country-r news channel" onek boro scope. Prothome 5-10 ta boro source (global + Bangladesh + kichu major country) diye shuru, pore baranor system rakho.
- Live-stream tracking (YouTube live etc.) alada, heavy kaj. Phase 2-e rakha bhalo.

## 7. NLP and Sentiment Pipeline

1. **Clean:** URL, emoji, duplicate remove, language detect (Bangla/English).
2. **Spam / fake filter:** Account age, post frequency, duplicate pattern, bot-like behavior, ar keyword-based scam pattern diye `spam_score`. Score beshi hole `is_flagged`, sentiment-e weight 0.
3. **Sentiment score:** FinBERT (English) ar multilingual model (Bangla-er jonno). Output: -1 theke +1.
4. **Relevance:** Text ta ei coin-er kina, embedding similarity ba coin name/ticker matching diye.
5. **Source weight:** `sources.weight` (known analyst, exchange report, major news beshi weight; unknown user kom).
6. **Reply/comment reaction:** Post-er replies-er sentiment alada score kore post score-er sathe milano.
7. **4h aggregate:**

`score = sum(sentiment x source_weight x relevance x (1 - spam_score)) / sum(weights)`

**Note:** Weight gulo hand-pick korle research-e question uthbe. Weight value ablation study diye justify korte hobe (weight sହ vs weight chara result tulona).

## 8. Machine Learning (ML)

**Role:** Baseline ar fast model. Tabular feature-er upor XGBoost.

**Features:**

- Price: returns, volatility, RSI, MACD, moving averages, volume change
- Sentiment: 4h score, shift, mention count, per-platform score
- Macro: rate, dollar index, ei dhoroner indicator
- Time: hour-of-day, day-of-week

**Target:** Next 4h / 1d / 7d return (regression) ar direction (classification, secondary).

**Models:** XGBoost (main baseline), Ridge/SVR (simple baseline, Paper 4-er sathe tulonar jonno).

## 9. Deep Learning (DL)

| Model | Kaj |
| --- | --- |
| GRU | Fast sequence baseline |
| LSTM | Mockup-e ache, paper comparison-er jonno |
| Transformer encoder | Long dependency, cross-asset attention (optional, time thakle) |

**Input:** Last N candle (jemon 60 ta 4h candle) + sentiment features, per asset.

**Transfer learning (new coin-er jonno):**

1. Boro coin (BTC, ETH, etc.) shob mile ekta shared model pretrain.
2. Notun coin-er kom data diye fine-tune (kom epoch, low learning rate).
3. Coin-er ID embedding add kora jate model coin-er pothik porichoy jane.
4. Evaluation: kom data-r coin-e fine-tuned vs scratch-e trained, tulona.

## 10. Training and Update Strategy

**Expanding window, 4h cycle:**

```
Every 4 hours:
  1. Fetch new price, news, social data
  2. Update features
  3. Predict next horizon with current model
  4. When actual result comes, record error
  5. Add new data to training set (expanding window)
  6. Fine-tune / retrain on schedule (not necessarily every 4h)
```

**Important:** Prottek 4h-e full retrain na kore, daily/weekly full retrain + 4h-e shudhu inference ar light update. Naile compute onek beshi lagbe, ar noise-ey overfit hote pare.

**Data leakage rules (khub important):**

- Random train/test split kora jabe na. Shudhu time-based split ba walk-forward validation.
- Feature normalization shudhu train data theke fit kore test-e apply.
- News/social-er timestamp theke future data kono moteo input-e dhukbe na. Post-er `ts` ar candle close-er ts mila-e dekhte hobe.

## 11. Uncertainty and Evaluation

**Uncertainty:** Quantile regression (10%, 50%, 90%) ba conformal prediction diye lower/upper range. Frontend-e band hishebe dekhano hobe.

**Prediction metrics:**

| Metric | Meaning |
| --- | --- |
| RMSE / MAE | Price error |
| SMAPE | Percentage error (mockup-e ache) |
| Directional accuracy | Direction thik hoyeche kina |
| Interval coverage | 80% range-e actual kotobar porlo (80%-er kachakachi hote hobe) |

**Trading metrics (backtest):**

| Metric | Meaning |
| --- | --- |
| Net return | Fee + slippage-er por |
| Sharpe ratio | Risk-adjusted return |
| Max drawdown | Sobcheye boro poton |
| Win rate, trade count | Strategy behavior |

**Backtest assumptions:** Fee 0.1% per trade (mockup-er sathe mil), slippage assumption (jemon 0.05-0.1%), ar slippage-er sensitivity test (0%, 0.05%, 0.1%, 0.2%).

**Baselines tulonar jonno:** Buy-and-hold, naive (kal-er price = aj-er price), ar paper-er model (SVR).

## 12. Vision Agent (AI Assistant)

**Role:** User-er coin niye proshno-r uttor, briefing, portfolio audit, alert/post draft.

**Design:**

- LLM tool-calling pattern (mockup-er moto-i). Agent kono price nijer theke bolbe na, tool theke nebe.
- **Tools:** `get_market`, `get_forecast`, `get_sentiment`, `get_portfolio`, `propose_rebalance`, `list_alerts`, `get_social_pulse`. Write tool (`create_alert`, `post_community`) shudhu user "Autonomous" on korle.
- Agent kokhono real trade ba fund move korte parbe na.
- Prottek tool call log hobe ar frontend-er "Agent activity" panel-e dekhabe.
- System prompt-e bolte hobe: forecast probabilistic, financial advice na.
- Prompt injection: news/social text agent-er instruction na, data. Oi text-e likha command agent follow korbe na.
- Rate limit per user.

## 13. API Endpoints

Frontend document-er contract-er sathe mil rekhe:

| Endpoint | Kaj |
| --- | --- |
| `POST /auth/signup`, `/auth/login`, `/auth/logout` | Auth (JWT, httpOnly cookie) |
| `GET /markets` | Pair list, price, 24h change |
| `WS /stream/prices` | Live price |
| `GET /forecast?asset=&horizon=&model=` | Point, lower, upper, metrics, last\_updated, next\_run |
| `GET /sentiment?asset=` | Score, shift, per-platform, flagged count |
| `GET /news?source=` | Items with score ar weight |
| `POST/GET/DELETE /orders` | Paper trading |
| `GET /wallet`, `POST /wallet/transfer`, `/wallet/convert` | Wallet |
| `GET/POST /community/posts`, `POST /community/posts/{id}/vote` | Community |
| `GET /backtest?asset=&strategy=` | Backtest result |
| `POST /agent/chat` | Vision Agent |

**Response-e `last_updated` ar `next_run` thakbe, jate frontend 4-hour cycle indicator dekhate pare.**

## 14. Security

- JWT access token (short-lived) + refresh token, httpOnly cookie.
- Password: argon2/bcrypt, rate-limited login, email validation.
- CORS shudhu frontend domain-er jonno.
- Sob input Pydantic diye validate. SQL injection-er jonno ORM / parameterized query.
- API key (exchange, news, social) `.env`-e, kokhono Git-e na. `.gitignore`-e `.env` rakhte hobe.
- User-generated content (post, comment) server-e sanitize, ar upload file size/type limit.
- Paper trading-e real fund nai, kintu user data-r security real-er moto-i treat korte hobe.

## 15. Deployment

- `docker-compose`: api, postgres (timescale), redis, scheduler/worker, mlflow.
- Dev: local Docker. Demo: single VPS ba cloud VM. GPU lagbe shudhu DL training-e, inference CPU-te cholbe.
- Model file versioned (MLflow), prottek forecast-e `model_version` save hobe.
- Logging + basic monitoring (error rate, job failure, data freshness).

## 16. Build Phases

**Phase 1 · Foundation**

- [ ] Repo, Docker, FastAPI skeleton, config, logging
- [ ] PostgreSQL + TimescaleDB, Alembic, core tables
- [ ] Auth (signup/login, JWT)

**Phase 2 · Data**

- [ ] Price ingestion (CCXT), 10-20 coin historical backfill
- [ ] News + 2-3 social source ingestion
- [ ] 4h scheduler

**Phase 3 · Sentiment**

- [ ] Clean, spam filter, FinBERT + multilingual
- [ ] Source weight, relevance, 4h aggregate

**Phase 4 · Models**

- [ ] Feature pipeline, walk-forward validation
- [ ] XGBoost + SVR baseline
- [ ] GRU/LSTM, shared model + transfer learning for new coin

**Phase 5 · Evaluation**

- [ ] Uncertainty (quantile/conformal)
- [ ] Backtest with fee + slippage
- [ ] Ablation: with/without sentiment, with/without source weight

**Phase 6 · App services**

- [ ] Forecast, sentiment, markets, wallet, orders, community APIs
- [ ] WebSocket prices
- [ ] Vision Agent

**Phase 7 · Integration and polish**

- [ ] Frontend connect, CORS, auth flow
- [ ] Monitoring, tests, documentation

## 17. Risks

| Risk | Mitigation |
| --- | --- |
| Social API access/cost/terms | Official API only, prothome free source, source list dhire baranor |
| Scope onek boro | #1, #2, #3 main, #4 simple version, baki optional |
| Data leakage | Walk-forward validation, timestamp check |
| New coin-e kom data | Transfer learning, kom epoch fine-tune |
| Noisy crypto market | Uncertainty band dekhano, overclaim na kora |
| Compute (4h retrain) | Full retrain daily/weekly, 4h-e inference |
| LLM hallucination | Tool-grounded answer, no invented price |

## 18. Notes

- Crypto price predict kora kothin, tai research-e "model profit dibe" dabi na kore "kon feature (sentiment, weight) forecast improve kore" ei ta prove kora beshi safe ar honest.
- Ei document-er research contribution tomar confirm-er opekkhay, gap choice bodlale Section 1, 7, 9, 11 update korte hobe.
- Frontend document-er sathe API contract mil rekhe change korte hobe, duita-i ek sathe update.
