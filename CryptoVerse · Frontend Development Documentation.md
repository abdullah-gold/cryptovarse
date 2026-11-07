# CryptoVerse · Frontend Development Documentation

## 1. Overview

CryptoVerse ekta crypto forecasting ar paper-trading web app. Ei document-e mockup (single HTML file) theke real React frontend banar plan, structure ar rules deya ache.

**Goal:** Login-er age simple public pages, login-er pore full features (Trade, Wallet, Dashboard, Agent, Community).

**Scope:** Shudhu frontend. Backend (FastAPI + ML) alada document-e thakbe. Prothome frontend mock/simulated data diye chalbe, pore API connect hobe.

## 2. Tech Stack

| Kaj | Tool |
| --- | --- |
| Framework | React 18 + Vite |
| Language | TypeScript (recommended) |
| Routing | React Router v6 |
| State | Zustand (global), React Query (server data) |
| Charts | lightweight-charts (candlestick), Recharts (line/area) |
| Styling | CSS Variables + CSS Modules (mockup-er design token reuse hobe) |
| Forms | React Hook Form + Zod |
| Realtime | WebSocket (native) |
| Testing | Vitest + React Testing Library |
| Lint/Format | ESLint + Prettier |

## 3. Folder Structure

```
src/
  app/            router, providers, layout shell
  assets/         fonts, icons, images
  components/     shared UI (Button, Card, Chip, Modal, Toast, Table)
  features/
    auth/         login, signup, auth store, ProtectedRoute
    markets/      market list, search, sparkline
    trade/        chart, order book, order form, orders table
    wallet/       overview, spot/funding, earn, deposit/withdraw
    dashboard/    forecast chart, signal inputs, watchlist
    sentiment/    social pulse, live feed, headline scorer
    community/    posts, polls, alerts, comments
    agent/        Vision Agent chat, mission control
    pricing/      plans, FAQ
  hooks/          useTheme, usePrices, useWebSocket
  lib/            api client, formatters, constants
  styles/         tokens.css, global.css
  types/          shared TypeScript types
```

## 4. Routing and Auth Gate

| Route | Page | Access |
| --- | --- | --- |
| `/` | Home (exchange-style) | Public |
| `/markets` | Markets list | Public (preview), Trade button locked |
| `/features` | Features tabs | Public |
| `/pricing` | Plans + FAQ | Public |
| `/contact` | Contact form | Public |
| `/login`, `/signup` | Auth | Public |
| `/trade/:symbol` | Trade screen | Login required |
| `/wallet` | Wallet | Login required |
| `/dashboard` | Forecast dashboard | Login required |
| `/agent` | Vision Agent | Login required |
| `/community` | Community feed | Login required |

**Rules:**

- `ProtectedRoute` component auth state check kore. Login na thakle `/login?next=<path>`-e pathay.
- Locked feature-e click korle "Login to continue" prompt (modal) dekhabe, shudhu redirect noy.
- Login-er por `next` path-e ferot pathabe.
- Token: shuru-te memory + httpOnly cookie (backend ready hole). Mock phase-e localStorage chalbe, kintu real token localStorage-e rakha jabe na.

## 5. Design System

Mockup-er token gulo `styles/tokens.css`-e CSS variable hishebe jabe.

**Dark theme (default):**

| Token | Value | Use |
| --- | --- | --- |
| `--bg` | `#0a0c0f` | Page background |
| `--s1` | `#12151a` | Card surface |
| `--s2` | `#1a1e25` | Input, raised surface |
| `--ln` | `#272c34` | Border |
| `--tx` | `#ebe7df` | Text |
| `--mu` | `#8e939c` | Muted text |
| `--ac` | `#f0b34a` | Accent |
| `--up` | `#5fd3b0` | Positive / Buy |
| `--dn` | `#ff7a6b` | Negative / Sell |
| `--bl` | `#8fb4ff` | Info |

**Light theme:** `bg #f6f3ec`, `s1 #ffffff`, `s2 #efe9dc`, `ln #e0d8c7`, `tx #1d1b17`, `mu #6a655a`, `ac #d98a00`, `up #0c9b74`, `dn #d6433a`.

**Fonts:** Fraunces (headings), Manrope (body), JetBrains Mono (numbers, prices).

**Theme rule:** `html.light` class toggle, preference save hobe `cv-theme` key-te. Nav position (top/bottom) ar "Toggle animations" (reduced motion) setting-o save hobe.

**Responsive breakpoints:** 860px (mobile layout, bottom tab bar), 1260px (nav collapse). `prefers-reduced-motion` respect korte hobe.

## 6. State Management

| Store | Kaj |
| --- | --- |
| `authStore` | user, token, login/logout |
| `uiStore` | theme, navPosition, animations on/off, toast |
| `marketStore` | prices, 24h change, favorites |
| `tradeStore` | selected symbol, order form, open orders, history |
| `walletStore` | Spot/Funding/Earn balances, transactions |
| `communityStore` | posts, votes, ratings, comments |
| `agentStore` | messages, tool trace, mode, autonomous toggle |

Server data (forecast, sentiment, news) React Query diye fetch ar cache hobe, Zustand-e duplicate rakha jabe na.

## 7. Key Components

**Shared:** `Button`, `Chip`, `Card`, `Modal`, `BottomSheet`, `Toggle`, `Toast`, `DataTable`, `Sparkline`, `PriceChange` (green/red pill), `CoinAvatar`.

**Feature components:** `CandleChart`, `OrderBook`, `OrderForm`, `OrdersTable`, `ForecastChart`, `SentimentMeter`, `SocialPulseCard`, `NewsFeed`, `PostCard`, `PostComposer`, `PollOption`, `AgentChat`, `ToolTrace`, `AllocationBar`, `PlanCard`.

**Layout:** `Header`, `NavBar` (top/bottom), `Footer`, `PageShell`.

## 8. API Contract (planned)

Backend ready na howa porjonto mock service (`lib/mockApi.ts`) use hobe. Same function signature rakhte hobe, jate pore shudhu implementation bodlano jay.

| Endpoint | Kaj |
| --- | --- |
| `POST /auth/login`, `/auth/signup`, `/auth/logout` | Authentication |
| `GET /markets` | Pair list, price, 24h change |
| `WS /stream/prices` | Live price push |
| `GET /forecast?asset=&horizon=&model=` | Forecast + error metrics (SMAPE, direction hit-rate) |
| `GET /sentiment?asset=` | Score, 3-day shift, per-platform pulse |
| `GET /news?source=` | Live feed items with score |
| `POST /orders`, `GET /orders`, `DELETE /orders/:id` | Paper trading |
| `GET /wallet`, `POST /wallet/transfer`, `/wallet/convert` | Wallet actions |
| `GET/POST /community/posts`, `POST /community/posts/:id/vote` | Community |
| `POST /agent/chat` | Vision Agent |

Sob response-er type `types/` folder-e define hobe.

## 9. Build Phases

**Phase 1 · Setup**

- [ ] Vite + React + TypeScript project
- [ ] ESLint, Prettier, folder structure
- [ ] tokens.css, global.css, theme toggle
- [ ] Router, layout shell (Header, NavBar, Footer)

**Phase 2 · Public pages**

- [ ] Home
- [ ] Markets (preview)
- [ ] Features, Pricing, Contact
- [ ] Login, Signup (form validation shoho)

**Phase 3 · Auth gate**

- [ ] authStore, ProtectedRoute
- [ ] Locked-feature prompt, `next` redirect

**Phase 4 · Core app**

- [ ] Trade (chart, order book, order form, orders)
- [ ] Wallet (overview, accounts, earn, deposit/withdraw/transfer/convert)
- [ ] Dashboard (forecast vs actual, signal toggles, watchlist)

**Phase 5 · Social ar AI**

- [ ] Social pulse, live feed
- [ ] Community (post, poll, alert, vote, comment)
- [ ] Vision Agent chat

**Phase 6 · Backend integration ar polish**

- [ ] mockApi bad diye real API
- [ ] WebSocket live prices
- [ ] Loading, empty, error state shob jaygay
- [ ] Accessibility, performance, testing

## 10. Research Gaps jeta UI-te add korte hobe

Mockup-e ei feature gulo nei, kintu research-er jonno dorkar:

1. **All / new cryptos:** Forecast shudhu 3 coin-e limited. Coin list dynamic hobe, "New" badge shoho.
2. **Source weighting:** Well-known people (influencer) er post-er weight ar fake/spam account flag dekhano.
3. **4-hour cycle:** "Last updated" ar "Next run in" indicator.
4. **Risk ar cost:** Fee, slippage, backtest result, forecast uncertainty band.

## 11. Coding Conventions

- Component: PascalCase file. Hook: `useXxx`. Feature folder-er baire feature-specific code import kora jabe na.
- Inline style minimize, CSS Modules + tokens use korte hobe.
- Number formatting `lib/format.ts` theke (price, percent, volume), duplicate formatter likha jabe na.
- Prottek async UI-te loading, empty, error state thakbe.
- User input (post, comment) React diye render hobe (default escape). `dangerouslySetInnerHTML` use kora jabe na. Mockup-e `innerHTML` use hoyechilo, eta real app-e carry kora jabe na.
- Commit message: `feat:`, `fix:`, `refactor:`, `docs:`.
- Branch: `main` + `feature/<name>`.

## 12. Setup Commands

```
npm create vite@latest cryptoverse-web -- --template react-ts
cd cryptoverse-web
npm i react-router-dom zustand @tanstack/react-query react-hook-form zod lightweight-charts recharts
npm i -D vitest @testing-library/react eslint prettier
npm run dev
```

## 13. Notes

- Mockup-er sob data simulated, real app-er footer-e "not financial advice" notice rakhte hobe.
- Vision Agent mockup-e `window.claude` er upor depend kore. Real app-e eta backend `/agent/chat` endpoint diye replace hobe.
- Mockup file-er shesh part (Wallet-er por) review kora hoyni, Phase 4-er age oi part dekhe nite hobe.
