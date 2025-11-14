requirements.md

Overview

Build shrd, a web app that lets users save content from across the web into Collections, which can be formatted as long-form articles or listicles and shared publicly. Users authenticate with Google, add URLs that become embeds or Open Graph link cards, organize them, and share them. The app supports RSS ingestion, bulk adding content, ad placements, and a Chrome extension for quick saving.

This first pass focuses on core collection creation, URL ingestion, public rendering, feed importing, and the Chrome extension. Advanced features like newsletters or WordPress syncing are not included in this phase.

⸻

Tech Stack (Suggested) • Frontend: Next.js 14 with React • Styling: Tailwind CSS • Backend: Next.js API routes or Node backend • Database: Postgres (via Prisma or Drizzle) • Auth: Google OAuth (NextAuth recommended)

The generator may choose reasonable alternatives.

⸻

Design and UX Guidelines • Overall look • Light background, high contrast, generous whitespace. • Sans-serif font (system UI stack or Inter). • Navigation • Simple top nav with logo/name left, user profile right. • Layout • Fully responsive: single column on mobile, optional sidebar on desktop. • Components • Clear primary button per view. • Simple, non-bloated forms. • Empty states • Dashboard: “Create your first Collection”. • New Collection: “Paste a URL to begin”. • Feedback • Use simple toasts or banners for success and errors.

⸻

Developer Experience • Provide .env.example containing: • DATABASE_URL • GOOGLE_CLIENT_ID • GOOGLE_CLIENT_SECRET • NEXTAUTH_SECRET • Scripts: • npm run dev • npm run build • npm run start • npm run db:migrate • Include README.md with: • Setup instructions • How to configure Google OAuth • How to run migrations

⸻

Security and Safety Requirements • Never insert untrusted HTML into the DOM without sanitization. • All metadata fetching and scraping must happen on the backend, not the client. • Limit: • Request timeouts • Maximum HTML size fetched • Sanitize embed HTML server side. • Respect robots.txt when possible. • Provide clear error messages for fetch failures.

⸻

Error Handling and Empty States • API must return JSON error bodies: { error: string }. • Frontend should: • Display inline form errors • Use alerts or toasts for network errors • Metadata fetch failure: • Allow adding URL anyway with minimal item data • Feed failure: • Return friendly error • Do not block Collection editing

⸻

Analytics Hooks (Basic)

Add placeholders so implementation can be expanded later. Calls can be stubs.

Events to track: • collection_viewed • url_added • item_clicked

Function signature can be simple:

trackEvent(eventName, payload)

⸻

Primary User Flow (Happy Path) 1. User visits / and clicks “Sign in with Google”. 2. Redirected to /dashboard showing list of Collections. 3. User creates new Collection. 4. User pastes a YouTube URL, sees an embedded video appear. 5. User adds a blog URL, sees link card using Open Graph metadata. 6. User views public Collection page at /c/{slug}. 7. Page displays top ad container and responsive layout. 8. User imports a feed and bulk-adds recent posts.

⸻

Core Application Requirements

Authentication • Google OAuth sign-in. • Store minimal user info: • id, email, name, avatar_url, created_at. • Logged-in session reused for API access. • Endpoint: GET /api/me.
⸻

Collections
Schema

id owner_id title description is_public layout_type ("article" | "listicle") ordering ("manual" | "ascending" | "descending") slug created_at updated_at

Behaviors • Users can create/edit/delete Collections. • Public URL: /c/{slug}. • Layouts: • article: stacked sections • listicle: numbered items • Ordering: • manual uses the stored position field • ascending or descending sorts numerically

⸻

Collection Items
Schema

id collection_id source_url provider_type (youtube, tiktok, instagram, reddit, x, og, etc) embed_html (nullable) og_title og_description og_image_url position created_at

URL Handling Logic 1. Detect embed-capable platforms: • YouTube, TikTok, Instagram, Reddit, X, others via oEmbed. 2. If embed supported: • Retrieve embed HTML or iframe snippet • Sanitize server side 3. Else: • Fetch page server side • Parse Open Graph metadata • Fallback to <title> if OG missing

Link Card Format • Thumbnail image (if available) • Title • Description • Source domain

⸻

Public Collection Pages
Requirements: • Light, clean layout • Top ad container: • Desktop: 728x90 • Mobile: 320x50 or 300x250 • Responsive rendering down to 320px width • Automatic Open Graph tags for Collections: • og:title • og:description • og:image (first item or default)

Share buttons: • Copy link • X/Twitter • Facebook • LinkedIn

⸻

Feed Ingestion (Bulk Add)
Behavior 1. User pastes a site or section URL. 2. System tries to discover RSS/Atom feed links. 3. If RSS exists: • Use it 4. If not: • Perform simple HTML scraping to extract article links 5. Fetch feed items: • Max 5 items per site • Max 50 items total • Last 24 hours if timestamps available 6. Show items to user for selection 7. On confirm, each selected becomes a CollectionItem

FeedSource Schema

id owner_id source_url feed_url feed_type ("rss" | "generated") last_fetched_at

⸻

Chrome Extension (Save to shrd)

Build a Google Chrome extension that lets users save the current page to a Collection.

Goals • One-click “Save to shrd” from any webpage. • Reuse existing backend API for creating CollectionItems. • Use web session cookies rather than separate OAuth.

Core UX

Popup when clicking the extension icon

If not authenticated: • Show: “Sign in to shrd” • Button: “Sign in with Google” • Opens https://www.shrd.co/login in new tab

If authenticated (GET /api/me): • Show current tab URL • Dropdown listing user’s Collections • Most recent Collection preselected • Button: “Create new Collection” • Opens web UI in new tab • Primary button: “Add to Collection” • On success: “Added to {CollectionName}” with link to view it

Context menu (optional first pass) • Right-click → “Save to shrd…” • Opens popup with URL pre-filled (link if clicked; else page)

Auth Behavior • The extension relies on session cookies from the main web login. • All API calls use fetch with credentials: "include".

Required Backend Endpoints for Extension • GET /api/me • GET /api/collections?mine=1 • POST /api/collections/:id/items Body:

{ "url": "<current_tab_url>" }

Extension Technical Requirements • Manifest V3 • Minimum permissions: • "activeTab" • "storage" • "https://www.shrd.co/*" • Files: • manifest.json • popup.html + popup.js (or React bundle) • Optional background service worker for context menus

⸻

API Endpoints (First Pass) • GET /api/me • GET /api/collections?mine=1 • POST /api/collections • PUT /api/collections/:id • DELETE /api/collections/:id • POST /api/collections/:id/items • GET /api/collections/:id/items • POST /api/fetch-metadata • Input: { url: string } • Output: { provider_type, embed_html?, og_title?, og_description?, og_image_url? } • POST /api/feed/ingest • Input: { url: string } • Output: list of feed candidates with titles, URLs, timestamps.

⸻

Database Schema (Complete First Pass)

User

id email name avatar_url created_at

Collection

id owner_id title description is_public layout_type ordering slug created_at updated_at

CollectionItem

id collection_id source_url provider_type embed_html og_title og_description og_image_url position created_at

FeedSource

id owner_id source_url feed_url feed_type last_fetched_at

⸻

Out of Scope for This First Pass

Do not build yet: • WordPress MCP integration • Email newsletters or subscriber system • Payment plans • Multi-ad-unit layouts • Discovery/explore page • Analytics dashboard • Advanced scraping/crawling

These will be added in later phases.

⸻

Definition of Done (Minimum) 1. Users can sign in with Google. 2. Users can create/edit/delete Collections. 3. Users can add URLs that become embeds or link cards. 4. Public Collections render cleanly at /c/{slug} with ad container. 5. Responsive layout works on desktop and mobile. 6. Feed ingestion works for RSS and basic scraping. 7. Chrome extension can save the current page to a Collection. 8. Codebase includes env example, scripts, and README.
