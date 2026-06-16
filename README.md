# P2P-video-Calling



# Video Call App (WebRTC + Socket.io)

A simple peer-to-peer video calling web app. Two users join a shared "room" and connect directly to each other using WebRTC, with a lightweight Socket.io server handling the signaling (room joins, call offers/answers, and renegotiation).

## Tech Stack

- **Client:** React, Socket.io-client, native WebRTC (`RTCPeerConnection`)
- **Server:** Node.js, Socket.io
- **Signaling:** STUN servers (Google + Twilio public STUN) for ICE candidate discovery

## Project Structure

```
.
├── client/                     # React frontend
│   ├── public/
│   └── src/
│       ├── context/
│       │   └── SocketProvider.jsx   # Shares one Socket.io connection app-wide via React Context
│       ├── screens/
│       │   ├── Lobby.jsx            # Entry screen — enter email + room number to join
│       │   └── Room.jsx             # Actual call screen once a room is joined
│       ├── service/
│       │   └── peer.js              # Wraps RTCPeerConnection: creates offers/answers
│       ├── App.js
│       └── ...
└── server/                     # Signaling server
    ├── index.js                 # Socket.io server — relays join/call/negotiation events
    └── package.json
```

## How It Works

1. A user enters their email and a room number on the **Lobby** screen and submits the form.
2. The client emits a `room:join` event to the server with `{ email, room }`.
3. The server adds the socket to that room, notifies existing members via `user:joined`, and confirms the join back to the requester via `room:join`.
4. On confirmation, the client navigates to `/room/:roomId`.
5. Inside the room, peers exchange WebRTC **offers** and **answers** through the server (`user:call`, `incomming:call`, `call:accepted`) to negotiate a direct connection.
6. If media settings change mid-call (e.g. adding video), `peer:nego:needed` / `peer:nego:final` events handle renegotiation.
7. Once negotiation completes, audio/video/data flows **directly between the two browsers** — the server is only used for the initial handshake, not for relaying call data.

## Getting Started

### Prerequisites
- Node.js installed
- Two browser tabs/devices to test a call

### 1. Start the server
```bash
cd server
npm install
node index.js
```
The signaling server runs on **port 8000**.

### 2. Start the client
```bash
cd client
npm install
npm start
```
The React app runs on the default port (usually **3000**).

### 3. Test a call
- Open the app in two browser tabs.
- Join the **same room number** with two different emails.
- Once both are in the room, initiate a call from one tab.

## Notes

- CORS is enabled on the server so the client (on a different port) can connect.
- STUN servers help peers discover their public network info; no TURN server is configured, so calls behind very restrictive NATs/firewalls may fail to connect directly.
- `PeerService.setLocalDescription(ans)` is a slightly misleading method name — internally it calls `setRemoteDescription`, since it's used to store the **answer received from the other peer**, not this peer's own description.
