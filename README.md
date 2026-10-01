# TradePilot — MT5 Remote Trading Controller (Windows prototype v1.3)

Demo-first remote control system inspired by the uploaded pink trading keypad.

## Included

- FastAPI backend
- JWT login/register
- device registration + pairing tokens
- WebSocket command/result channel
- command IDs for idempotency
- audit log
- rate limiting
- emergency stop
- Windows MT5 Connector using the MetaTrader5 Python package
- React/Vite mobile PWA
- BUY / SELL / CLOSE / CANCEL / LIMIT
- lot +/-
- symbol, SL, TP and limit-price inputs
- live searchable symbol list loaded from the connected MT5 terminal
- compact active-session, local-time, and MT5 connection header
- automatic one-time setup-QR renewal in the Windows connector
- audible and animated feedback for primary trading commands
- safely spaced quick access to close winning or losing positions
- configurable Shark Buy/Sell grids with 1–3 lines, four editable lots, broker-point spacing, and Hedging-only SHARK BOTH
- separate Windows Market Lab opened beside the connector, with TradingView
  reference charts, strategy presets, and M1/M5/M15/H1/H4 controls
- Market Lab Pro chart-style, timezone, fullscreen, and saved-preference
  controls
- QR pairing UI
- Docker Compose
- Windows setup scripts
- architecture/security/MT5 documentation
- smoke tests

## Important

This is configured for DEMO trading by default. It does not include MetaTrader 5, broker credentials, or any broker account. Install MT5 separately and log into a Demo account first.

Do not disable the demo guard until you have independently tested the whole system.

## Quick start

### Server

Requirements: Python 3.11+

```powershell
cd server
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### Connector

Run on the Windows PC where MT5 is installed:

```powershell
cd connector
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env
python connector.py
```

The connector creates `connector_state.json` on first launch. Keep it private.
It now remembers a successfully claimed pairing token, so normal connector
restarts do not try to claim the same one-time token again.

### Web/PWA

```powershell
cd web
npm install
copy .env.example .env
npm run dev -- --host 0.0.0.0
```

Set `VITE_API_URL` to the PC's LAN address, for example `http://192.168.1.20:8000`.
Also add the PWA origin (for example `http://192.168.1.20:5173`) to
`ALLOWED_ORIGINS` in `server/.env`.

Open the Vite address from the phone while both devices are on the same LAN.

## First-use flow

1. Start MT5 and log into a Demo account.
2. Start the server.
3. Start the connector.
4. Register/login in the web app.
5. In the web app, create a one-time pairing token. Put that token in `connector/.env` as `PAIRING_TOKEN` and restart the connector. The connector registers itself and claims the token automatically.
6. Select the device.
7. Use a small Demo lot such as 0.01.
8. Test BUY, SELL, CLOSE and then the other functions.

The displayed QR encodes the one-time token, but v1.1 still requires copying
that token into the connector `.env`. Camera scanning inside the Windows
connector is not implemented yet.

## Docker (server and web only)

The MT5 connector must still run directly on the Windows MT5 PC.

```powershell
copy .env.docker.example .env
# Set a strong SECRET_KEY, the LAN PWA origin, and the LAN API address.
docker compose up --build
```

For a phone, do not leave `VITE_API_URL` as `127.0.0.1`; use the server PC's
LAN address. Rebuild the web image after changing this build-time value.

## Production hardening before real money

Use HTTPS/WSS, PostgreSQL, secret management, 2FA/passkeys, device approval/revocation, server-side exposure/loss limits, signed Windows builds, a Windows service, broker-specific filling-mode handling, penetration testing, and long-duration disconnect/reconnect tests.

Never port-forward the development server directly to the public internet.

`ALLOW_LIVE_TRADING=false` is the connector's independent live-account lock.
Do not change it until the full release checklist and an independent security
review are complete.

## Windows all-in-one installer build

Version 1.3 includes an all-in-one Windows desktop prototype. It starts the
local FastAPI server, serves the PWA, runs the MT5 Connector, displays a
one-time setup QR, adds a private-network firewall rule, and starts with
Windows.

Because the MetaTrader5 Python package contains a Windows-only native module,
the installer must be compiled on Windows. Double-click:

```text
BUILD_INSTALLER.bat
```

The script builds the PWA, packages the Windows application, installs Inno
Setup through `winget` if necessary, and produces:

```text
release\TradePilotSetup.exe
```

The development installer is unsigned, so Windows SmartScreen may warn on
first launch. Production distribution requires a code-signing certificate.
