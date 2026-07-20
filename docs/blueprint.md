# CryptoPing — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

CryptoPing is a Telegram bot that lets users maintain private crypto watchlists with price threshold and percentage-change alerts. It delivers on-demand price queries, optional morning digests, quiet hours suppression, and provides the owner with aggregated usage/analytics stats while preserving user privacy.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual crypto traders/holders
- Bot owner/administrator

## Success criteria

- Users receive accurate price alerts based on defined rules
- Owner can view aggregated alert statistics via /admin_stats
- Morning digest is delivered according to user preferences

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding and main menu
- **/price** (command, actor: user, command: /price) — Query current prices for watchlist or specific ticker
- **Add Bitcoin** (button, actor: user, callback: watchlist:add:BTC) — Quick-add Bitcoin to watchlist
- **Add Ethereum** (button, actor: user, callback: watchlist:add:ETH) — Quick-add Ethereum to watchlist
- **Add Toncoin** (button, actor: user, callback: watchlist:add:TON) — Quick-add Toncoin to watchlist
- **Custom Coin** (button, actor: user, callback: watchlist:add:custom) — Enter custom ticker symbol

## Flows

### Onboarding
_Trigger:_ /start

1. Explain features
2. Request timezone
3. Offer quick-add buttons for common coins

_Data touched:_ User profile

### Alert Management
_Trigger:_ watchlist entry creation

1. Select alert type (price threshold/percent change)
2. Configure parameters (direction, value, lookback)
3. Set cooldown interval

_Data touched:_ Watchlist entry

### Morning Digest
_Trigger:_ User-defined local time

1. Generate summary of 24h price movements
2. Apply quiet hours suppression rules
3. Send formatted digest message

_Data touched:_ Price snapshot, User profile

### Admin Analytics
_Trigger:_ /admin_stats

1. Fetch aggregated user count
2. Retrieve top tickers/alert types
3. Display non-identifying metrics

_Data touched:_ Alert event

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — User preferences and settings
  - fields: Telegram user ID, Display name, Timezone, Quiet hours (start/end), Digest time, Alert cooldowns, Notification preferences
- **Watchlist entry** _(retention: persistent)_ — Monitored crypto assets with alert rules
  - fields: Ticker symbol, Display name, Price threshold rules, Percent-change rules, Enabled status, Cooldown timestamps
- **Price snapshot** _(retention: persistent)_ — Historical price data for percent-change calculations
  - fields: Timestamp, Ticker, Price value
- **Alert event** _(retention: persistent)_ — Anonymized alert records for analytics
  - fields: Event type, Ticker, Direction, Percent change, Timestamp

## Integrations

- **Telegram** (required) — Bot API messaging, inline keyboards, and admin commands
- **Market Price API** (required) — Fetch current and historical crypto prices
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /admin_stats command to view aggregated metrics

## Notifications

- Price threshold alerts with old/new prices
- Percent-change alerts with direction/percentage
- Morning digest with 24h summary
- Snooze/adjust buttons for active alerts

## Permissions & privacy

- User data is stored persistently but never shared
- Admin stats show only aggregated non-identifying metrics
- Unknown tickers are confirmed before adding

## Edge cases

- Price API failures with retry logic
- Ambiguous tickers with suggested matches
- Quiet hours overlapping with alert triggers

## Required tests

- End-to-end alert triggering with cooldown handling
- Morning digest delivery during/after quiet hours
- Admin stats display of top tickers/alert types

## Assumptions

- Price check interval fixed at 5 minutes
- Percent-change lookback window fixed at 1 hour
- Default fiat currency is USD
