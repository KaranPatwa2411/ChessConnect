# ChessConnect (Chess.com clone - minimal)

Lightweight WebSocket-based chess demo with a TypeScript/Node backend and a React + Vite frontend.

This repository contains two folders:

- `backend1/` — Node WebSocket server (TypeScript, built with esbuild, uses `ws` and `chess.js`).
- `frontend/` — React + Vite client (TypeScript, Tailwind CSS) that connects to the backend over WebSocket.

## Quick summary

- Backend runs a WebSocket server on ws://localhost:8080 and pairs two connected clients into a `Game`.
- Frontend runs with Vite (default port 5173) and uses a small `ChessBoard` component and `useSocket` hook to connect.

## Repository layout (important files)

- `backend1/package.json` — backend scripts and dependencies.
- `backend1/src/index.ts` — starts WebSocket server on port 8080.
- `backend1/src/GameManager.ts` — pairs clients and routes messages.
- `backend1/src/Game.ts` — game logic using `chess.js`.
- `backend1/src/messages.ts` — message type constants.

- `frontend/package.json` — frontend scripts and deps.
- `frontend/src/hooks/useSocket.ts` — WebSocket connection (WS_URL: `ws://localhost:8080`).
- `frontend/src/screens/Game.tsx` — handles incoming messages and emits `init_game`.
- `frontend/src/components/ChessBoard.tsx` — UI for the board (move UI).

## Prerequisites

- Node.js (recommended v18+).
- npm (bundled with Node) or a compatible package manager.

Tested on Windows (PowerShell) in this workspace.

## Install and run (local development)

Open two terminal windows (one for backend, one for frontend):

Backend (server):

1. cd into the backend folder

   cd backend1

2. Install dependencies

   npm install

3. Start the server

   npm run dev

The `dev` script will build the TypeScript with `esbuild` and start `dist/index.js`. The server listens on ws://localhost:8080.

Frontend (client):

1. cd into the frontend folder

   cd frontend

2. Install dependencies

   npm install

3. Start the Vite dev server

   npm run dev

By default Vite will serve the app at http://localhost:5173 (check console output if using a different port).

Opening two browser windows (or two different browsers) to the frontend and clicking "Play" in each window will pair the two clients into a game.

## Available scripts

Backend (`backend1/package.json`):

- `npm run build` — bundle `src/index.ts` to `dist/index.js` using esbuild.
- `npm start` — run Node on `dist/index.js`.
- `npm run dev` — convenience (build then start).
- `npm run lint` / `npm run lint:fix` — ESLint checks for backend TypeScript.

Frontend (`frontend/package.json`):

- `npm run dev` — start Vite development server.
- `npm run build` — compile TypeScript project references and build the production bundle.
- `npm run preview` — preview the production build locally.
- `npm run lint` — run ESLint in the frontend.

## WebSocket protocol

Message types are defined in `backend1/src/messages.ts` and mirrored in the frontend (`frontend/src/screens/Game.tsx`):

- `init_game` — sent by the frontend to request a game. No payload.
- `move` — move message sent between server and clients while the game is running.
  - payload shape (example): `{ from: "e2", to: "e4" }`
- `game_over` — sent by server when chess.js detects game over. Payload contains `winner`.

Server pairing/flow (implementation notes):

- The backend stores a `pendingUser`. When a client sends `init_game`, if there's a pending user the server creates a `Game` and notifies both players of their colors. Otherwise the client becomes the pending user.
- The `Game` class enforces turn order and broadcasts `move` messages to the opponent. When `chess.js` reports game over, the server sends `game_over` to both clients.

Files to inspect for protocol and behavior:

- `backend1/src/GameManager.ts`
- `backend1/src/Game.ts`
- `frontend/src/hooks/useSocket.ts`
- `frontend/src/screens/Game.tsx`

## Configuration

- The frontend WebSocket URL is in `frontend/src/hooks/useSocket.ts` as `WS_URL` — change if you run the backend on a different host or port.
- The backend currently listens on port 8080 (see `backend1/src/index.ts`).

## Troubleshooting

- If the frontend cannot connect to the backend, check that the backend process is running and that ws://localhost:8080 is reachable.
- If port 8080 is in use, change the port in `backend1/src/index.ts` and update `WS_URL` in the frontend hook.
- If you modify backend TypeScript, re-run `npm run dev` (build+start) or run the `build` script before `npm start`.

## Development notes and TODOs

- Add validation (zod) for inbound messages on the backend before applying moves (the Game file has a comment about this).
- Add better handling for client disconnects (currently `removeUser` is a stub).
- Add persistence / spectating / matchmaking improvements.
- Add tests for Game logic and message handling.

## License & author

Author: Karan Patwa

This project is provided as-is for learning/demonstration purposes.
