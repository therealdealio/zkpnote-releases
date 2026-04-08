# Changelog

All notable changes to ZKPnote will be documented in this file.

## v0.3.0 — 2026-04-08

**Per-Note Proof of Originality (Major)**
- Complete smart contract redesign: vault-level hashing replaced with per-note SHA-256 proof of originality
- New on-chain program `zkpnote` (renamed from `chainnotes_vault`) deployed to devnet
- New program ID: `Ad67RwgTaeh77UQ5oZXAwt3fTvg3u5oNxNfcc3tGJLbc`
- `register_proof` instruction — stores SHA-256(title + content) in a PDA seeded by the hash
- First-to-register wins — if the PDA already exists, no one else can claim that content
- Prove button (purple shield) in note editor toolbar, turns green when proved
- Proof result banner with Solana Explorer link after proving
- Green shield icon next to proved notes in sidebar

**Public Verification Page**
- New `/verify` page — anyone can paste content and check if it's been proved on Solana
- Client-side SHA-256 hashing (nothing sent to servers until verify clicked)
- Exact match lookup via Supabase `proofs` table
- Fuzzy similarity search via PostgreSQL `pg_trgm` extension
- Full timestamps with seconds shown on verification results
- Verify links added to marketplace nav bar and sidebar

**Phantom Wallet Session Persistence**
- Phantom `signMessage` signature cached in sessionStorage
- No more Phantom popup or logout on page refresh
- Session survives browser refresh while tab is open

**Bug Fixes**
- Fixed proof data (proofTx, proofHash, purchasedFrom) being silently dropped during encrypt/decrypt cycle
- Fixed proof recovery when proof exists on-chain but local data was lost
- Fixed invalid Explorer link showing `/tx/recovered` — now correctly links to PDA address
- Fixed verify page not scrolling and results not showing

**Infrastructure**
- Supabase `proofs` table with trigram indexes for similarity search
- `search_similar_proofs` PostgreSQL function
- New `/api/proof` API route (store, verify, search actions)
- Priority fees (10,000 microLamports) added for devnet/mainnet transactions

## v0.2.0 — 2026-04-07

**Backend Migration**
- Migrated all API routes (vault, marketplace, share) from filesystem storage to Supabase
- Solana wallet address now stored alongside auth keypair in vault records

**Rich Text Editor**
- New WYSIWYG rich text editing mode powered by Tiptap
- Toggle between Markdown and Rich Text modes (persisted preference)
- Full formatting toolbar: headings, bold, italic, underline, strikethrough, highlight, lists, blockquote, code blocks, links, text alignment, undo/redo
- Notes stored as markdown internally — fully compatible with existing notes and marketplace

**Editor Improvements**
- Dark/light theme toggle for the editor area (sidebar stays dark)
- Light mode CSS for markdown preview and rich text editor
- Theme preference persisted to localStorage

**Sharing & Sending**
- Separated "Share" (read-only link) and "Send" (wallet-to-wallet delivery) into distinct features
- Share generates a read-only link with optional NDA
- Send delivers a copy to a specific Solana wallet address
- Auth signature now required for share creation

**Import/Export**
- Import markdown (.md) files directly as notes
- Import multiple files at once
- Export individual notes as .md files
- Export all notes as individual .md files
- Existing JSON vault backup import/export preserved

**Marketplace**
- Listing previews now render full markdown instead of raw text
- Sticky bottom buy bar for long listings
- Inline transaction confirmation with fee breakdown
- Repeatable purchases (buy button stays accessible after purchase)
- HTML rendering support via rehype-raw

**Branding**
- New logo: shield + Z mark representing protection and zero-knowledge proof
- "Zero-Knowledge Proof" label on landing page
- Renamed from ChainNotes to ZKPnote across exports

**Infrastructure**
- Deployed to Vercel (zkpnote.vercel.app)
- Supabase for persistent storage
- Solana devnet for blockchain operations
- Security headers (X-Frame-Options, X-Content-Type-Options, etc.)
- Rate limiting on all API endpoints

### Landing Page
- New "Access Anywhere" feature pillar — cross-platform access with just a seed phrase and internet connection
- Removed "Verified Identity" pillar (not yet implemented)
- New shield + Z logo replacing generic document icon across the app
- "Zero-Knowledge Proof" label added under brand name
- Updated tagline: "Your Ideas · Your Proof · Your Market"

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
