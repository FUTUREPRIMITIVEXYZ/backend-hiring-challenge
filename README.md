# FP Backend Challenge - Nounberg Terminal

Build a service that tracks Nouns DAO auctions, enriches each auction event with extra data, generates human-readable headlines for every auction, and makes those headlines available both in real time and through a paginated historical feed.

## Goal

* Index auction events from the Nouns `AuctionHouse` contract.
* Enrich each event with off‑chain data (USD price, ENS names, thumbnail URL).
* Generate and store a concise human-readable headline that summarizes the auction.
* Stream new headlines over WebSocket.
* Offer a paginated API so clients can scroll through past headlines.

## What to Build

### 1. Indexer
* Use **Ponder** to listen for `AuctionCreated`, `AuctionBid`, and `AuctionSettled` events from the Nouns `AuctionHouse` contract (`0x830BD73E4184cef73443C15111a1DF14e495C706`) on Ethereum mainnet .

### 2. Job Queue and Workers
* When an event arrives, enqueue a lightweight job using any queue system you prefer
  (BullMQ, RabbitMQ, SQS, Postgres LISTEN/NOTIFY, etc.).
* Worker responsibilities:
  * Convert ETH amounts to USD using a public price API.
  * Resolve ENS names for all addresses involved.
  * Fetch or compose a thumbnail URL for the Noun.
  * Create a human-readable headline that clearly summarizes the auction event (e.g. `Noun #721 sold for 69.42 Ξ ($248 000) to vitalik.eth`).
  * Assemble the all auction data and persist it in the database.

### 3. Data Persistence
* Store data in the database of your choice
* Store, at minimum:
  * Block & transaction metadata
  * Event type (`AuctionCreated`, `AuctionBid`, `AuctionSettled`)
  * Noun ID, price in ETH and USD, winner / bidder addresses & ENS
  * Thumbnail URL
  * Human-readable headline
  * Timestamps
* Ensure inserts are idempotent so chain re‑organisations or restarts never create duplicates.

### 4. Real‑Time Delivery
* Provide a realtime subscription endpoint (e.g. using websockets, SSE, etc) that broadcasts each new record as soon as it’s stored.

### 5. Paginated API
* Expose an HTTP endpoint that returns stored headline records in pages (newest first).
* Clients must be able to retrieve and display the headline and data for every item.
* Implement efficient pagination that performs well even with large datasets.

### 6. Developer Setup
* Supply a `docker-compose.yml` (or equivalent) that starts:
  * Database
  * Queue backend
  * Ponder indexer
  * API server / realtime gateway
  * Any other required services
* A single command should bring the full stack up for local testing.

## Bonus Points

* Cache ENS and price lookups with a sensible TTL.
* Back‑fill the last *N* auctions on startup so the feed isn’t empty.
* Health‑check and metrics endpoints for monitoring.
* Sign‑In with Ethereum (EIP‑4361) so only authenticated users can subscribe to realtime events.
* A minimal front end application that visualises the live feed.
* Tests that fork mainnet locally (e.g., with Anvil) and simulate an entire auction cycle.

## Deliverables

* **GitHub repository**
  * Include clear setup instructions and document any notable design decisions.

* **Quick demo video** (2–3 minutes)
  * Start the stack, show an auction or simulated event, and highlight:
    * A headline being stored in the database
    * The realtime service pushing the new headline
    * The paginated API returning headlines

Good luck!
