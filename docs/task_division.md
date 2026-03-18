# Campus Navigation System — Task Division

**Project:** Campus Navigation System for Schools
**Timeline:** 4 Days
**Team:** Feranmi (Backend), Nicasio (Database), Jaleel (Frontend), Moyin (Backend Support + Group Lead)

---

## Why Moyin Is Paired with Backend

After reviewing the system design doc, the **backend** carries the heaviest workload:

- **Backend:** Routing engine (Dijkstra/A*), 15+ API endpoints, authentication, search logic, moderation workflows, realtime features, and event-aware navigation logic.
- **Database:** Schema creation, PostGIS setup, seeding, indexing, and full-text search configuration — significant but well-scoped.
- **Frontend:** Map rendering, search UI, route display, event pages, admin dashboard — substantial but parallelizable with component-based architecture.

Moyin joins Feranmi on the backend to split the routing engine and API surface.

---

## Team Assignments

### Feranmi — Backend (Core API + Auth + Moderation)

**Responsibilities:**
- Set up FastAPI project structure, config, and middleware
- Implement authentication (school email auth, session management)
- Build the following API endpoints:
  - `GET /places/search?q=` — place search with fuzzy matching
  - `GET /places/:id` — place detail
  - `GET /events?query=` — event search
  - `GET /events/:id` — event detail
  - `POST /events` — event creation
  - `POST /reports` — user report submission
  - `POST /map-edits` — user map edit submission
  - `GET /admin/review-queue` — moderation queue
  - `POST /admin/review/:id/approve` — approve/reject edits
- Implement moderation workflow logic (review queue, approval, versioning)
- Rate limiting and input validation
- WebSocket setup for realtime closure/event push (stretch)

### Moyin — Backend (Routing Engine + Search)

**Responsibilities:**
- Design and implement the routing engine:
  - Graph construction from `path_nodes` and `path_edges` tables
  - Dijkstra's algorithm for correctness
  - A* optimization for performance
  - Weight calculation per route mode:
    - **Fastest:** `weight = est_walk_seconds`
    - **Safer-at-night:** `weight = est_walk_seconds + lighting_penalty + isolation_penalty + closure_penalty`
    - **Accessible:** `weight = est_walk_seconds + stair_penalty + inaccessibility_penalty`
  - Dynamic weight adjustments (time of day, temporary closures)
- Build the routing API endpoint:
  - `POST /routes` — accepts start/end (place ID or coordinates), mode (fastest/safer/accessible), and time context (day/night)
- Implement the search layer:
  - PostgreSQL full-text search integration
  - Trigram matching for fuzzy/alias search
  - Support for official names, nicknames, abbreviations, and categories
- Implement event-aware navigation logic:
  - Link events to venue entrances
  - "Leave now to arrive by X" time calculation
- Help Feranmi with integration testing across all endpoints

### Nicasio — Database

**Responsibilities:**
- Set up PostgreSQL with PostGIS extension
- Create all tables from the schema design:
  - `places` (id, official_name, display_name, aliases[], category, lat, lng, building_id, description, is_active)
  - `path_nodes` (id, lat, lng, node_type, metadata)
  - `path_edges` (id, from_node_id, to_node_id, geometry, distance_meters, est_walk_seconds, is_accessible, has_stairs, lighting_score, safety_score, popularity_score, open_start_time, open_end_time, status, confidence_score, source, updated_at)
  - `events` (id, title, description, venue_place_id, start_time, end_time, organizer, tags[], visibility)
  - `user_reports` (id, user_id, report_type, referenced_place_id, referenced_edge_id, payload, status, created_at)
  - `map_edits` (id, proposed_by, edit_type, object_type, object_id, diff_payload, review_status, reviewer_id, created_at)
  - `temporary_closures` (id, object_type, object_id, reason, start_time, end_time, severity)
- Set up proper indexes:
  - GiST indexes on geometry columns for spatial queries
  - GIN indexes on `aliases` array and `tags` array columns
  - Trigram index (`pg_trgm`) on `official_name`, `display_name`, and `aliases` for fuzzy search
  - Full-text search index on place names
  - B-tree indexes on foreign keys and frequently queried columns
- Seed the database with initial campus data:
  - Buildings, entrances, landmarks, bus stops, gates
  - Known paths, sidewalks, shortcuts, crosswalks
  - Edge metadata (distances, accessibility flags, lighting scores)
  - Sample events for testing
- Write database migration scripts
- Create database utility functions/stored procedures for:
  - Spatial queries (nearest node lookup, radius search)
  - Graph edge retrieval for routing engine
- Set up connection pooling configuration

### Jaleel — Frontend

**Responsibilities:**
- Set up Next.js project with routing structure
- Integrate MapLibre GL JS with OpenStreetMap base layers
- Build the following pages/components:
  - **Map View (main page):**
    - Campus map rendering with zoom/pan
    - Campus-specific overlays (buildings, paths, landmarks)
    - Route polyline display on map
    - Current location marker
  - **Search:**
    - Search bar with autocomplete/suggestions
    - Search results list with place details
    - "Navigate here" action from search results
  - **Route Selection:**
    - Route mode selector (Fastest / Safer at Night / Accessible)
    - Start and end point selection (search or tap on map)
    - Display ETA and route summary
    - Step-by-step route directions panel
  - **Events:**
    - Event listing page with search/filter
    - Event detail view with "Navigate to Event" button
    - "Leave by" time display
  - **User Reports:**
    - "Report a problem" form (missing place, wrong name, closed path)
    - "Suggest a path" submission form
  - **Admin Moderation Dashboard:**
    - Review queue list view
    - Edit diff viewer
    - Approve/reject actions
- Responsive design for mobile web
- API integration with backend endpoints
- Loading states, error handling, and empty states

---

## 4-Day Schedule

### Day 1 — Foundation

| Person | Tasks |
|--------|-------|
| **Nicasio** | PostgreSQL + PostGIS setup, create all tables and indexes, begin seeding campus data |
| **Feranmi** | FastAPI project scaffolding, auth implementation, places endpoints (`GET /places/search`, `GET /places/:id`) |
| **Moyin** | Routing engine graph construction, Dijkstra implementation, weight calculation functions |
| **Jaleel** | Next.js setup, MapLibre GL JS integration, base map rendering, search bar component |

### Day 2 — Core Features

| Person | Tasks |
|--------|-------|
| **Nicasio** | Finish seeding, write spatial query utilities, full-text + trigram search setup, migration scripts |
| **Feranmi** | Event endpoints, report/map-edit submission endpoints, moderation queue endpoints |
| **Moyin** | A* optimization, `POST /routes` endpoint, route mode weighting (fastest/safer/accessible), dynamic factors |
| **Jaleel** | Search results UI, route display on map, route mode selector, start/end point selection |

### Day 3 — Integration + Secondary Features

| Person | Tasks |
|--------|-------|
| **Nicasio** | Connection pooling, query optimization, support backend team with data issues, edge case data fixes |
| **Feranmi** | Rate limiting, input validation, moderation workflow logic, WebSocket setup (stretch) |
| **Moyin** | Search layer (full-text + trigram integration), event-aware navigation ("leave by" logic), integration testing |
| **Jaleel** | Event listing + detail pages, user report forms, API integration for search + routing |

### Day 4 — Polish + Testing + Launch Prep

| Person | Tasks |
|--------|-------|
| **Nicasio** | Final data validation, seed data completeness check, database backup setup |
| **Feranmi** | End-to-end API testing, bug fixes, API documentation |
| **Moyin** | Cross-team integration testing, routing edge case testing, help with any blockers across the team |
| **Jaleel** | Admin dashboard, responsive design pass, error/loading states, final UI polish |

---

## Group Lead Check-In Checklist

Use this checklist at the end of each day to confirm every teammate is on track.

### Day 1 Check-In

#### Nicasio (Database)
- [ ] PostgreSQL instance is running and accessible by the team
- [ ] PostGIS extension is enabled and verified
- [ ] All 7 tables are created (`places`, `path_nodes`, `path_edges`, `events`, `user_reports`, `map_edits`, `temporary_closures`)
- [ ] GiST, GIN, and trigram indexes are in place
- [ ] Initial campus seed data is loaded (at least buildings and major paths)
- [ ] Team can connect to the database from their local environments
- **Blocker check:** Any issues with PostGIS installation or spatial data formats?

#### Feranmi (Backend)
- [ ] FastAPI project runs locally with a health check endpoint
- [ ] Auth flow works (signup/login with school email, token issued)
- [ ] `GET /places/search?q=` returns results from seeded data
- [ ] `GET /places/:id` returns a single place with full details
- [ ] API connects to Nicasio's database successfully
- **Blocker check:** Any issues with database connectivity or auth library setup?

#### Moyin (Backend — Self-Check)
- [ ] Graph construction loads nodes and edges from database correctly
- [ ] Dijkstra's algorithm produces a valid shortest path on test data
- [ ] Weight functions are implemented for all 3 modes (fastest, safer, accessible)
- [ ] Unit tests pass for routing with simple test graphs
- **Blocker check:** Is the graph structure from the database sufficient, or does Nicasio need to adjust?

#### Jaleel (Frontend)
- [ ] Next.js app runs locally
- [ ] MapLibre GL JS renders the campus base map (OpenStreetMap tiles)
- [ ] Map supports zoom, pan, and centering on campus coordinates
- [ ] Search bar component is built and triggers search on input
- [ ] Project folder structure is organized (components, pages, utils)
- **Blocker check:** Any issues with MapLibre setup or tile rendering?

---

### Day 2 Check-In

#### Nicasio (Database)
- [ ] Full campus seed data is loaded (buildings, entrances, paths, shortcuts, landmarks, sample events)
- [ ] Edge metadata is populated (distances, walk times, accessibility flags, lighting scores)
- [ ] Spatial query utility works (e.g., "find nearest node to these coordinates")
- [ ] Full-text search returns results for place names
- [ ] Trigram search returns results for partial/fuzzy input (e.g., "libr" matches "Library")
- [ ] Migration scripts are documented and reproducible
- **Blocker check:** Is the seed data realistic enough for routing and search testing?

#### Feranmi (Backend)
- [ ] `GET /events?query=` returns filtered events
- [ ] `GET /events/:id` returns event with linked venue info
- [ ] `POST /events` creates a new event successfully
- [ ] `POST /reports` accepts a user report and stores it
- [ ] `POST /map-edits` accepts a map edit proposal and stores it
- [ ] `GET /admin/review-queue` returns pending edits/reports
- [ ] `POST /admin/review/:id/approve` changes edit status
- **Blocker check:** Any issues with request validation or database writes?

#### Moyin (Backend — Self-Check)
- [ ] A* algorithm works and is faster than Dijkstra on the campus graph
- [ ] `POST /routes` endpoint accepts start/end + mode + time context
- [ ] Fastest mode returns shortest-time route
- [ ] Safer mode applies lighting/isolation penalties correctly
- [ ] Accessible mode avoids stairs and inaccessible edges
- [ ] Dynamic factors (time of day, closures) adjust weights correctly
- [ ] Route response includes: path coordinates, ETA, distance, steps
- **Blocker check:** Is the edge metadata complete enough for safer/accessible routing?

#### Jaleel (Frontend)
- [ ] Search bar sends requests to `/places/search` and displays results
- [ ] Tapping a search result shows the place on the map
- [ ] Route polyline renders on the map between two points
- [ ] Route mode selector (Fastest / Safer / Accessible) is functional
- [ ] User can select start and end points via search
- [ ] ETA and distance are displayed for a computed route
- **Blocker check:** Any issues with API response format or map polyline rendering?

---

### Day 3 Check-In

#### Nicasio (Database)
- [ ] Connection pooling is configured and tested under concurrent requests
- [ ] Query performance is acceptable (routes compute in < 2 seconds)
- [ ] Any data gaps identified by backend team are fixed
- [ ] Edge cases in spatial data are handled (e.g., disconnected nodes, zero-distance edges)
- **Blocker check:** Any slow queries or missing data blocking other team members?

#### Feranmi (Backend)
- [ ] Rate limiting is active on public endpoints
- [ ] Input validation rejects malformed requests with clear error messages
- [ ] Moderation workflow works end-to-end: submit edit -> appears in queue -> approve -> data updates
- [ ] Approved map edits are versioned and reversible
- [ ] WebSocket connection is established (stretch — at minimum, closure data is available via polling)
- **Blocker check:** Any unhandled error cases or security gaps?

#### Moyin (Backend — Self-Check)
- [ ] Search layer integrates full-text + trigram matching via Nicasio's indexes
- [ ] Search supports official names, aliases, nicknames, and abbreviations
- [ ] Event-aware navigation works: event -> venue -> route
- [ ] "Leave by" time calculation is accurate given ETA
- [ ] Integration tests pass for: search -> route -> navigate flow
- [ ] Helped resolve at least one cross-team integration issue
- **Blocker check:** Any mismatches between frontend expectations and API response formats?

#### Jaleel (Frontend)
- [ ] Event listing page renders events from API
- [ ] Event detail page shows venue info and "Navigate to Event" button
- [ ] Clicking "Navigate to Event" computes and displays a route
- [ ] "Report a problem" form submits successfully to `/reports`
- [ ] "Suggest a path" form submits successfully to `/map-edits`
- [ ] All search, routing, and event flows work end-to-end with live API
- **Blocker check:** Any API endpoints returning unexpected formats or errors?

---

### Day 4 Check-In (Final)

#### Nicasio (Database)
- [ ] All seed data is complete and validated (no orphan nodes, no broken edges)
- [ ] Database backup process is documented
- [ ] All indexes are verified and performing well
- [ ] No open data issues reported by other teammates
- [ ] Schema documentation is written (table purposes, column descriptions)
- **Ship-ready?** Can the database support a demo of the full app without errors?

#### Feranmi (Backend)
- [ ] All API endpoints tested end-to-end (happy path + error cases)
- [ ] No unhandled 500 errors on any endpoint
- [ ] Auth flow tested (signup, login, protected routes, invalid tokens)
- [ ] Moderation flow tested (submit, queue, approve, reject)
- [ ] API documentation is written (endpoint list, request/response examples)
- [ ] All bugs found during integration are fixed
- **Ship-ready?** Can someone hit every endpoint and get correct responses?

#### Moyin (Backend — Self-Check)
- [ ] Routing works for all 3 modes across the seeded campus graph
- [ ] Edge cases tested: same start/end, unreachable destination, no accessible path
- [ ] Search returns relevant results for at least 10 different query variations
- [ ] Event navigation flow works end-to-end
- [ ] Cross-team integration is verified: frontend -> backend -> database -> response
- [ ] Final round of testing complete with no critical bugs open
- **Ship-ready?** Does the full user flow work: search -> select -> route -> navigate?

#### Jaleel (Frontend)
- [ ] Admin moderation dashboard renders review queue and supports approve/reject
- [ ] App is responsive on mobile screen sizes
- [ ] Loading spinners show during API calls
- [ ] Error messages display when API calls fail
- [ ] Empty states are handled (no results, no events, no pending edits)
- [ ] Final UI polish pass complete (alignment, spacing, consistent styling)
- [ ] Full demo walkthrough works without errors
- **Ship-ready?** Can a new user open the app, search, route, and navigate without confusion?

---

## Quick Reference: Who Owns What

| Feature Area | Owner | Support |
|---|---|---|
| Database schema + setup | Nicasio | — |
| Data seeding + spatial queries | Nicasio | Moyin (data validation) |
| Auth + session management | Feranmi | — |
| Places + Events API | Feranmi | — |
| Reports + Moderation API | Feranmi | — |
| Routing engine | Moyin | — |
| Routing API endpoint | Moyin | Feranmi (integration) |
| Search layer | Moyin | Nicasio (indexes) |
| Event-aware navigation | Moyin | Feranmi (event API) |
| Map rendering | Jaleel | — |
| Search UI | Jaleel | — |
| Route display + mode selector | Jaleel | — |
| Event pages | Jaleel | — |
| User report/edit forms | Jaleel | — |
| Admin moderation dashboard | Jaleel | Feranmi (API) |
| Integration testing | Moyin | All |
