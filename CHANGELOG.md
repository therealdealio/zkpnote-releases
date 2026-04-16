# Changelog

All notable changes to ZKPnote will be documented in this file.

## v0.8.0 — 2026-04-16

**Web UX Overhaul** — major batch covering everything shipped 2026-04-14 → 2026-04-16, bringing the web app to parity with the desktop app and beyond.

### New — Discoverability & Navigation

- **Command palette (⌘K)** — unified launcher. Fuzzy filter notes by title or tag, run actions (New note, Quick capture, Prove, Sync, Toggle focus, Show shortcuts).
- **Cheat sheet (⌘/)** — overlay listing every keyboard shortcut, grouped.
- **Quick capture (⌘⇧N)** — modal for dumping a thought without leaving your current note. ⌘⏎ saves.
- **Tab bar** — multi-note editing. Drag to reorder, ⌘W close, ⌘⇧T reopen, ⌘1–9 jump, middle-click close. Persists across reload.
- **Status bar** — bottom strip showing lock state, sync age (click to force sync), on-chain proof, word count. Hover labels for plain-language explanations.
- **Breadcrumbs** above the title show the folder chain.
- **Sidebar Folders / Tags toggle** — flip between folder tree and a tag-grouped view.

### New — Editor

- **Find in note (⌘F)** — overlay with yellow highlights for all matches, orange ring for the active one. Enter cycles forward, Shift+Enter back, scroll-safe.
- **Wikilink autocomplete** — type `[[` and pick from a fuzzy dropdown of note titles.
- **Wikilink hover preview** — hover any rendered `[[link]]` for a 280ms popover with the target's title, tags, and first 400 characters. Click to jump.
- **Backlinks panel** — every note that links to the current one, plus every outgoing link.
- **Focus mode (⌘.)** — collapse all chrome. Esc or Mode ▾ → Default to restore.
- **Typewriter mode (⌥⌘T)** — caret stays vertically centered as you scroll.
- **Mode ▾ dropdown** in toolbar — visible switcher so focus mode is never a one-way trap.
- **Inline image paste** — paste any image (or use the toolbar button). Stored as an encrypted attachment in your vault, referenced with a short `zkp-attach:<id>` URL instead of a 100KB base64 blob.
- **Tiptap rich-text mode** now supports tables, images, and `[[wikilinks]]`. Round-trip with markdown preserves all of them, including GFM pipe-table syntax.
- **History panel** — auto-saves a revision every 10 seconds while you edit (last 50 per note). View any rev, restore with confirm. Local-only.
- **Save indicator** — Saving… / Saved / Save error in the meta bar.
- **Auto-theme** on first load follows your OS `prefers-color-scheme`.
- **⌘⏎** in the editor proves the active note on Solana.

### New — Sidebar

- **Multi-select** — click Select, check notes via Cmd-click or Shift-click range, then bulk **Export .md** (downloads a single zip of markdown files) or **Delete**.

### New — Settings (⌘⇧,)

- Theme (follow system / dark / light)
- Auto-sync delay
- Network info (Solana cluster, your wallet address)
- Export vault (encrypted JSON or all notes as `.md`)
- Lock vault
- Quick reference for all keyboard shortcuts

### New — Mobile

- Sidebar scrolls properly on mobile (was unbounded before).
- Wallet bar hidden on mobile to reduce clutter.
- Send / Sell toolbar buttons hidden on narrow viewports (Share stays).
- Status bar simplified on mobile.

### Improved — Prove flow

- Single progressive button: `Prove on Solana ↗` → `Waiting for Phantom…` → `On-chain ✓`.
- Auto-refresh on tab focus — switching tabs and back picks up newly finalized proofs.
- Sell on an unproved note now auto-runs the proof first instead of showing a hostile alert.
- Toast notifications for success and errors (with Retry).

### Fixed

- React hydration error caused by `<div>` descendants inside markdown `<p>` elements (WikilinkHover now uses `<span>` throughout).
- Find overlay no longer steals focus from the search input on every keystroke.
- Find overlay no longer rubber-bands the editor scroll position while you type.
- Find highlight layer wrap matches the textarea exactly.
- Skeleton loader now appears during session-recovery reloads, not just first unlock.
- Settings shortcut moved from ⌘, to ⌘⇧, (Chrome was hijacking ⌘,).
- Markdown tables no longer get mangled going through Rich text mode and back.
- Multi-file markdown export bundles into a single .zip (Chrome was silently blocking sequential downloads).

### Infrastructure

- Toast notification system with success / error / info kinds and optional action buttons.
- Top-level ErrorBoundary — a thrown error from any component now shows a recoverable fallback instead of white-screening.
- Shared Icon module to consolidate inline SVGs.
- Skeleton loader during vault unlock / decrypt.

### Testing

- 28-test Playwright bot covering load + hydration, every shortcut, find, image paste (markdown + rich text), table round-trip, mobile layout, and full keyboard interaction with no console errors.

## v0.7.0 — 2026-04-16

**Remote MCP Endpoint — Claude Mobile + claude.ai Web**

- New Next.js route at `/api/mcp` exposing a remote HTTP MCP transport (`WebStandardStreamableHTTPServerTransport` from `@modelcontextprotocol/sdk`) so Claude mobile (iOS / Android) and claude.ai on the web can connect to ZKPnote as a custom connector — previously blocked because both clients only support remote, not stdio, MCP servers.
- Tool handlers factored into `src/lib/mcpServer.ts` as `buildMcpServer({ seedPhrase, apiUrl })`, shared by the remote HTTP route. The existing stdio server in `packages/mcp-server/` is untouched and Claude Desktop / VSCode configs keep working unchanged.
- Crypto envelope is byte-identical between stdio and remote (BIP-44 `m/44'/501'/0'/0'` → deterministic signature of `"ChainNotes-vault-encryption-key-v1"` → first 32 bytes encryption key, last 32 bytes auth keypair). A note written from Claude mobile is the same ciphertext as one written from Desktop or the web UI.
- **Remote exposes 15 tools** — the full vault + proofs + knowledge-graph surface plus `browse_marketplace`. The three marketplace-detail tools (`get_listing`, `cancel_listing`, `marketplace_analytics`) remain stdio-only for this release.
- **Auth:** bearer-token gate (`Authorization: Bearer <ZKPNOTE_MCP_TOKEN>` or `?token=<...>` query param for mobile clients that can't set headers). Fail-closed: missing env var returns HTTP 500 rather than silently accepting anonymous requests.
- **Current deployment model:** single-tenant — one seed phrase + one shared-secret token per Vercel deployment. Multi-tenant per-user auth is the next roadmap item.
- **Endpoint URL is `https://www.zkpnote.com/api/mcp`** — always with `www.`. The bare `zkpnote.com` 307-redirects to `www`, and most HTTP clients drop `Authorization` across redirects.

**End-to-End Bot Test**

- New `scripts/test-mcp-bot.mjs` — 26 assertions exercising tool discovery, full CRUD cycle, batch save, folder creation, knowledge-graph discovery, and cleanup. Marker-isolated so the bot leaves the vault in its original state.
- Supports `VERCEL_BYPASS` env var to forward `x-vercel-protection-bypass` headers against preview deploys gated by Vercel Deployment Protection.

**Documentation**

- MCP Setup page rewritten with Option A (stdio) / Option B (remote) structure, self-hosting walkthrough, and Claude mobile + claude.ai connector steps.
- Architecture page now documents both transports side-by-side with a per-tool availability matrix.
- FAQ adds "Can I use ZKPnote with Claude on my phone?" plus the `www.` vs apex-domain gotcha.
- Security page adds a Remote MCP Endpoint section covering the bearer-token model, threat matrix additions, and what's not yet implemented (per-user auth, MCP-route rate limiting, audit log).
- Public `zkpnote-docs/mcp-server.md` and `zkpnote-help/mcp-setup.md` synced to match.

## v0.6.0 — 2026-04-13

**Vault Search**
- New search bar in the sidebar below the Notes header — filter notes by title or content in real time
- Case-insensitive matching across note titles and body content
- When searching, folder tree flattens to show all matching notes regardless of folder
- Clear button (X) to reset search; empty search restores normal folder view

**Bug Report System**
- Replaced nodemailer-based bug reporting with Supabase `bug_reports` table
- "Report a Bug" button in sidebar footer with modal textarea
- Sends wallet, username, user agent, and URL alongside user description
- Rate limited: 50/hour/IP

**Admin Panel — Bug Reports Tab**
- New Bug Reports tab alongside existing Accounts tab
- Table with time, user, action, error, status columns
- Expandable rows with full details (wallet click-to-copy, mode, URL, IP, UA)
- Resolve button per open bug; red badge shows open count
- Auto-refresh every 30 seconds

**Knowledge Graph Query Improvements**
- Scoring rewrite: title matches weighted 3× vs body 1×
- Minimum score threshold based on query term count
- Results capped at 10; dropped per-entry tags for compactness
- Token efficiency stats on MCP tool responses

**Wallet Bar**
- Added "Faucet" button linking to faucet.solana.com (devnet only, hidden on mainnet)

## v0.5.0 — 2026-04-10

**Wallet Account Tracking**
- Every wallet that unlocks ZKPnote — Phantom or seed phrase — is now auto-registered in the `accounts` table with a deterministic `SHA-256(wallet_address)`-derived username (`phantom_<hash>` or `user_<hash>`)
- Admin page now shows Phantom wallets and any account without an email, not just fully linked accounts
- One-shot backfill migration populates tracking rows for every existing vault (`supabase-migration-backfill-tracking.sql`)
- `encrypted_seed` column in `accounts` is now nullable so Phantom users can register without storing a seed phrase server-side

**Link Account for Phantom**
- Phantom users can now open **Profile → Link Account** to reserve a custom username + optional email without providing any seed phrase (previously required a seed)
- Profile modal pre-fills the email from the linked account when the vault profile has no email stored
- Removed the Change Password UI for Phantom-only accounts (there is no server-stored seed to re-encrypt)

**Share Link: Inline Sign-Up**
- Share link sign-up form now collects **username + email + password + verify password** (previously only email + password) and registers a real account via `/api/account`
- Readers who sign up via a share link can immediately log back in with their chosen username and password on the main app
- Removed the confusing "Create Your Own Vault" footer button from the shared-note reveal view — readers already have a wallet by the time they see the content

**Email Delivery**
- `/api/seed-token` now supports generic SMTP (e.g. Namecheap Private Email) via `SMTP_HOST` / `SMTP_USER` / `SMTP_PASS` / `SMTP_FROM` environment variables, with Gmail as an automatic fallback
- Lets production deploys send recovery emails from a branded address (e.g. `admin@zkpnote.com`) without relying on a personal Gmail

**MCP Server: Batch Save**
- New `save_notes` batch tool — save many markdown notes in a single call (one vault pull + one push for the whole set), much faster than repeated `save_note` calls
- Added a `run.sh` wrapper that rebuilds the MCP server before executing, preventing stale builds from overriding features like tombstone merge
- Tool total is now **16** (up from 15)

## v0.4.0 — 2026-04-08

**Cancel / Unlist Marketplace Listings**
- Sellers can now cancel their own marketplace listings via a two-step confirmation flow (Cancel → Confirm)
- For "original" listings (where the note was removed from the vault at listing time), the note content is automatically restored to the seller's vault upon cancellation
- Server validates seller ownership before deletion

**Drag-and-Drop Note Reordering**
- Notes in the sidebar can now be reordered by drag and drop within a folder
- Six-dot grip handle appears on hover for dragging
- Purple drop indicator line shows the insertion point
- Sort order persists via new `sortOrder` field on notes (backward-compatible — null falls back to updatedAt sorting)

**Pop-Out Floating Window**
- Notes can be popped out into a floating always-on-top window via the toolbar button
- Uses the Document Picture-in-Picture API (Chrome/Edge) for true always-on-top behavior
- Falls back to a standard popup window on Safari/Firefox
- Floating window includes title editing, content editing, and live word count
- Edits in the floating window sync back to the main vault in real time
- Auto-closes when switching notes

**Proof Recovery API**
- New `recover` action on `/api/proof` — retrieves full note content from a proof's tx signature (owner-only)
- New `list` action on `/api/proof` — lists all proofs for a given wallet address
- Enables recovery of note content even if the local vault copy is lost

**Proof-Based Similarity Detection**
- New 90% similarity threshold for on-chain proved content — blocks marketplace listings that are too similar to other authors' proved notes, even if that content was never listed for sale
- Existing marketplace similarity check remains at 70%
- Fixed: same-seller content was incorrectly bypassing similarity checks
- Fixed: short content (few words) was bypassing similarity entirely due to ILIKE pre-filter thresholds

**MCP Server**
- Fixed `sortOrder` not being preserved through encrypt/decrypt cycle — MCP vault pushes no longer strip drag-and-drop ordering from notes

**UI / UX**
- Added @zkpnote X (Twitter) follow link to welcome page footer
- Fixed welcome page not scrolling on smaller viewports

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

**Unified Wallet & Vault Access**
- Solana keypair now derived via BIP-44 (`m/44'/501'/0'/0'`) — same address as Phantom and all standard Solana wallets
- Seed phrase and Phantom login now produce the same encryption key, auth keypair, and vault
- Log in with 12 words or Phantom and see the same notes — seamless switching between methods
- Phantom `signMessage` signature cached in sessionStorage — no popup on refresh

**Marketplace: Proof Required Before Listing**
- Notes must be proved on-chain before they can be listed on the marketplace
- Three layers of enforcement: UI alert, client-side validation, and server-side API rejection
- Establishes provenance before any commercial activity — sellers prove authorship first

**Built-In Documentation Site**
- New `/docs` route with Obsidian-style sidebar navigation and dark-theme markdown rendering
- 8 pages: Getting Started, FAQ, Marketplace Guide, Architecture, Smart Contract, Marketplace API, Security, White Paper
- All sidebar and welcome page links now point to built-in docs instead of GitHub repos

**Welcome Page & Branding**
- Animated electric shield logo on the welcome page (lightning arcs, spark particles, pulsing glow)
- Alpha Testing badge (amber pill with pulsing dot, "Alpha Testing · Devnet")
- ZKPnote shield favicon added (replaced default Next.js/Vercel assets)

**Bug Fixes**
- Fixed key derivation mismatch: seed phrase mode previously derived a different Solana address and encryption key than Phantom, creating separate vaults for the same wallet
- Fixed proof data (proofTx, proofHash, purchasedFrom) being silently dropped during encrypt/decrypt cycle
- Fixed proof recovery when proof exists on-chain but local data was lost
- Fixed invalid Explorer link showing `/tx/recovered` — now correctly links to PDA address
- Fixed verify page not scrolling and results not showing

**MCP Server for Claude & AI Agent Integration**
- Built `@zkpnote/mcp-server` package with 8 tools: `save_note`, `list_notes`, `read_note`, `update_note`, `delete_note`, `list_folders`, `verify_content`, `search_similar`
- Uses `@modelcontextprotocol/sdk` with stdio transport
- Derives encryption keys from `ZKPNOTE_SEED_PHRASE` env var — agents interact with your real vault
- Default API URL: `https://zkpnote.com`

**Infrastructure**
- Supabase `proofs` table with trigram indexes for similarity search
- `search_similar_proofs` PostgreSQL function
- New `/api/proof` API route (store, verify, search actions)
- Priority fees (10,000 microLamports) added for devnet/mainnet transactions
- Replaced in-memory rate limiter with Upstash Redis (`@upstash/ratelimit`) for distributed rate limiting; falls back to in-memory for local dev
- Marketplace browse now returns paginated results (page/limit/total) with Previous/Next controls
- Replaced O(N) similarity check with targeted ILIKE queries for marketplace search
- Solana RPC proxy whitelist — only 11 permitted methods, returns 403 for others
- `packages/` excluded from Next.js TypeScript build (`tsconfig.json`)

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
