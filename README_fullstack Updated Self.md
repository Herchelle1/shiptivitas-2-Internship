# Shiptivity: Kanban Board (Full Stack)

A drag-and-drop Kanban board for a shipping productivity tool, now backed by a **Node.js/Express API and a SQLite database**. Cards can be moved between lanes or reordered within a lane, and their position is saved, so the board looks the same after a page refresh.

This is the full-stack update of [shiptivitas-1-Internship](https://github.com/Herchelle1/shiptivitas-1-Internship), the frontend-only version where card positions reset on refresh. It was built as Module 2 of the *Y Combinator: Working as a Software Engineer at a Start Up* job simulation on Forage, a virtual experience where you work on a hypothetical startup's shipping productivity tool.

## Features

- Three swimlanes: **Backlog**, **In Progress**, **Complete**
- Moving a card to another lane updates its status in the database
- Reordering a card inside a lane updates its priority in the database
- Card order and lane persist after a page refresh
- Every lane keeps unique priorities (1 = top), with no gaps or duplicates
- Input validation with clear error messages

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React, Dragula |
| Backend | Node.js, Express, CORS |
| Database | SQLite via Node's built-in `node:sqlite` module |

## Getting Started

**Prerequisites:** Node.js **22.5 or newer** (needed for the built-in `node:sqlite` module) and npm.

```bash
# 1. Clone the repository
git clone https://github.com/Herchelle1/shiptivitas-2-Internship.git
cd shiptivitas-2-Internship
```

**Start the backend** (runs on port 3001):

```bash
cd <backend-folder>
npm install
npm start        # or: node server.js
```

**Start the frontend** (runs on port 3000), in a second terminal:

```bash
cd <frontend-folder>
npm install
npm start
```

Open [http://localhost:3000](http://localhost:3000). The frontend calls the API at `http://localhost:3001`, so the backend must be running first.

## Data Model

The `clients` table in `clients.db`:

| Column | Type | Description |
|---|---|---|
| `id` | integer | Unique id of the card |
| `name` | text | Client name |
| `description` | text | Short description |
| `status` | text | `backlog`, `in-progress` or `complete` (null is treated as `backlog`) |
| `priority` | integer | Position in its lane, where 1 is the top |

## API Reference

Base URL: `http://localhost:3001`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/v1/clients` | List all clients, ordered by priority. Optional filter: `?status=backlog\|in-progress\|complete` |
| GET | `/api/v1/clients/:id` | Get one client by id |
| PUT | `/api/v1/clients/:id` | Change a client's status and/or priority |

### `PUT /api/v1/clients/:id`

Request body (both fields optional):

```json
{ "status": "in-progress", "priority": 2 }
```

- `status` must be `backlog`, `in-progress` or `complete`
- `priority` must be a positive integer
- Returns the **full list of clients** after the update

Errors:

| Code | When |
|---|---|
| 400 | Invalid id, status or priority (the response includes a `message` and `long_message`) |
| 500 | The update failed and was rolled back |

## How It Works

1. When a card is dropped, the frontend finds the lane it landed in and the card's position within that lane only.
2. It sends `PUT /api/v1/clients/:id` with the new `status` and `priority`.
3. The server runs a database **transaction**:
   - Gets the ids of the target lane, without the moved card
   - Inserts the moved card at the requested position
   - Renumbers the target lane 1..n, so no two cards share a priority
   - If the card changed lanes, renumbers the old lane to close the gap
4. The server returns the updated list of clients, and the frontend re-renders the three lanes from it.
5. On refresh, the frontend loads `GET /api/v1/clients`, which is ordered by priority, so the board looks the same as before.

Wrapping the updates in a transaction means a failed request cannot leave the board half-updated.

## Key Files

- `server.js`: Express API, SQLite access and the reordering logic
- `clients.db`: the SQLite database
- `Board.jsx`: main React component: loads clients, sets up Dragula, sends an update on every drop
- `Swimlane` component: renders one lane and its cards

## Acknowledgements

- Task and starter code from the Y Combinator job simulation on [Forage](https://www.theforage.com/)
- Drag-and-drop by [Dragula](https://github.com/bevacqua/dragula)
