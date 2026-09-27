# Meross for Raycast

Control Meross smart plugs and power strips from Raycast, built on the unofficial [`meross-cloud`](https://github.com/Apollon77/meross-cloud) library.

## Commands

| Command | Mode | What it does |
| --- | --- | --- |
| **Search Devices** | view | Lists all plugs (and each outlet of a power strip) with their on/off state. Toggle with ↵, copy UUID/IP, or create a quicklink you can bind to a hotkey. |
| **Toggle Device** | view | Pick an online device (type to filter) and toggle it with ↵; Raycast closes and shows a HUD. Optional argument pre-fills the filter. |
| **Switch Device by Name** | no-view | Switches a device by name without opening a window — used by hotkeys and the quicklinks created from Search Devices. Arguments: device name, action (`toggle` / `on` / `off`). |
| **Menu Bar Devices** | menu-bar | Shows how many plugs are on and toggles them from the menu bar. Refreshes every 10 minutes. |
| **Log out** | no-view | Ends the Meross cloud session and removes the stored token. |

## Setup

```bash
npm install
npm run dev
```

Raycast asks for your Meross email and password on first launch. The region (e.g. `iotx-eu.meross.com`) is picked automatically, because the Meross login redirects to it.

If your account uses two-factor authentication, open **Search Devices** once and enter the MFA code. The login token is stored in the extension's LocalStorage and reused, so later commands don't need a code again.

## How it works

- Each command opens one cloud session (HTTP login + MQTT), does its work, and disconnects. The token is reused so that no new login is created every time.
- With **Local Network** enabled (default), the first query learns each device's LAN IP (`innerIp`), stores it, and sends later commands straight to `http://<ip>/config`. If the device doesn't answer locally, the command goes through the cloud instead.
- Devices that report `Appliance.Control.ToggleX` are switched per channel. Older devices use `Appliance.Control.Toggle`. On power strips, channel 0 switches the whole strip.
- Only on/off switching is supported. Lights, shutters, thermostats etc. are listed as *Unsupported*.

## Caveats

- Meross has no official API. A firmware or cloud change can break this extension at any time.
- `meross-cloud` was last released in December 2023 and depends on the deprecated `request` package.
- `meross-cloud`'s `getTokenData()` hashes the regional domain, but when it reads the token back it compares against the default domain, so a stored token was never reused. `src/lib/meross.ts` fixes this by re-hashing the token before storing it.
- Set `author` in `package.json` to your Raycast username before you publish.
