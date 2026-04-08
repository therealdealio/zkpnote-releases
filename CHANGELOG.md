# Changelog

All notable changes to ZKPnote will be documented in this file.

## v0.1.0 — 2026-04-06

### Initial Release

**Core Vault**
- End-to-end encrypted note-taking with AES-256 encryption via libsodium
- 12-word BIP-39 seed phrase for vault creation and recovery
- Markdown editor with live preview (GitHub Flavored Markdown)
- Folder organization with nested hierarchy and drag-and-drop
- User profiles with username, bio, and profile picture
- Vault export/import as encrypted JSON backup files
- Auto-sync with cloud backup (encrypted at rest)

**Blockchain Integration**
- On-chain vault hash (SHA-256) written to Solana for tamper-proof verification
- Anchor smart contract deployed on Solana (program: `9AbLiwQ82manor3YyArrQhhpxPCFha5xbF187EtdDae5`)
- Automatic SOL funding for transaction fees (devnet/localnet)

**Wallet**
- Built-in Solana wallet derived from seed phrase
- Phantom wallet integration as alternative login method
- Send/receive SOL with transaction confirmation
- Real-time balance display
- Devnet airdrop for testing

**Marketplace**
- List notes for sale: Copies (unlimited), Original (one-time transfer), or Auction (timed bidding)
- Content preview modes: Masked (redacted structure), Partial (first N characters), Full (complete preview)
- Marketplace search and category filtering (Tutorial, Research, Template, Guide, Course, Recipe, Cheatsheet, General)
- Masked content preview shows note structure without revealing text
- Atomic on-chain payment splitting: 98% to seller, 2% marketplace fee to treasury
- Auction support with starting bid, reserve price, duration picker (1h to 7 days), live countdown, bid history, and winner settlement
- Similarity detection (trigram-based Jaccard similarity) blocks reselling of purchased content
- Purchased notes tagged and prevented from re-listing
- Marketplace analytics dashboard with transaction history and revenue stats

**Sharing**
- Share notes via link with optional NDA requirement
- Recipient can view shared content with wallet-signed NDA acknowledgment

**Security**
- All encryption/decryption happens client-side; server never sees plaintext
- Anchor program uses PDA-based access control with `has_one` constraints
- Checked arithmetic for fee calculations (overflow protection)
- Treasury address validated on-chain via constraint matching

### Known Limitations
- Auction bids are off-chain (no SOL locked during bidding); payment occurs at settlement
- Similarity detection uses word trigrams at 70% threshold; may not catch heavily rewritten content
- Phantom wallet integration requires a full wallet (not view-only)
