# Fashion Try-On Bot — Bot specification

**Archetype:** custom

**Voice:** friendly and helpful — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that lets users upload photos, try on outfits from a brand catalog or their own uploads, and generate static or animated previews. Users can save results and favorite outfits locally, with optional admin controls for catalog management.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual shoppers
- fashion enthusiasts

## Success criteria

- users can upload and try on outfits with 90% success rate
- weekly admin summaries are delivered with 100% reliability
- users can save and favorite outfits without errors

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with instructions and buttons for uploading photos, browsing catalog, etc.
- **Upload Photo** (button, actor: user, callback: upload:photo) — Initiates photo upload process with guidance checklist
- **Browse Catalog** (button, actor: user, callback: catalog:browse) — Shows available outfit options from brand catalog
- **Upload Outfit** (button, actor: user, callback: upload:outfit) — Allows users to upload their own outfit images
- **Favorites** (button, actor: user, callback: favorites:view) — Displays user's saved favorite outfits
- **/admin** (command, actor: admin, command: /admin) — Opens admin menu for catalog management (only visible to owner)
- **Try Another Outfit** (button, actor: user, callback: try:another) — Allows users to try different outfits in the same session

## Flows

### Photo Upload
_Trigger:_ upload:photo

1. User sends photo
2. Bot checks for face visibility
3. User confirms or adjusts crop
4. Photo is stored temporarily

_Data touched:_ Uploaded user photo

### Outfit Selection
_Trigger:_ catalog:browse

1. Show catalog options
2. User selects one or multiple outfits
3. Confirm selection

_Data touched:_ Outfit item

### Preview Generation
_Trigger:_ preview:generate

1. User selects preview type
2. Generate static or animated preview
3. Show preview with action buttons

_Data touched:_ Try-on session

### Favorite Management
_Trigger:_ favorites:view

1. Display favorites list
2. Allow removal or download
3. Update favorites list

_Data touched:_ Favorite tags

### Admin Catalog Management
_Trigger:_ /admin

1. Verify admin identity
2. Show catalog management options
3. Add/remove brand items

_Data touched:_ Outfit item

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with preferences and favorites
  - fields: user_id, favorites, last_active
- **Uploaded user photo** _(retention: persistent)_ — Single-person clothed photo for try-on
  - fields: photo_id, user_id, timestamp, metadata
- **Outfit item** _(retention: persistent)_ — Brand catalog entry or user-uploaded outfit
  - fields: outfit_id, source, image_url, timestamp
- **Try-on session** _(retention: session)_ — One photo + one or multiple outfit attempts
  - fields: session_id, user_id, photo_id, outfit_ids, timestamp
- **Favorite tags** _(retention: persistent)_ — Per-user favorite outfit collection
  - fields: favorite_id, user_id, outfit_id, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging
- **Admin Chat** (optional) — Weekly summaries and catalog management
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add brand catalog items via admin commands
- Configure image retention period
- Enable/disable animated previews
- View weekly usage stats and brand addition summaries

## Notifications

- Weekly admin summary with brand additions and usage stats
- User notifications for successful uploads and preview generation

## Permissions & privacy

- Images are stored temporarily and deleted after 30 days (configurable)
- Favorites are stored server-side but only accessible to the user
- No personal images are shared publicly or with third parties
- Admin only receives non-personal usage stats and brand addition summaries

## Edge cases

- User uploads photo without visible face
- User selects multiple outfits for try-on
- User tries to access favorites without any saved
- Admin attempts to add invalid catalog items
- Image processing fails due to poor quality

## Required tests

- End-to-end test of photo upload to preview generation
- Test admin catalog management commands
- Verify image retention and deletion after retention period
- Test favorites management (add, remove, download)

## Assumptions

- Users will follow the image guidance checklist
- Admin will manage catalog items manually via chat commands
- Image processing will work for most valid inputs
- Users will primarily use the bot for static previews
