# Webex Integrated Far End Camera Control

Integrate Cisco RoomOS Device camera control into the Webex application using the embedded browser.

## Demo
[![Vidcast Overview](https://github.com/user-attachments/assets/7773a087-489b-4687-826a-f37e4bfd8af3)](https://app.vidcast.io/share/f936e8ac-14ef-4bd4-bd28-1ddc995a2c73)

## What’s in this repo

- **`server.js`** — Node (Express) service that talks to [Webex REST](https://developer.webex.com/docs/getting-started): devices lookup, direct messages, room tabs, and [xAPI `Camera.Ramp`](https://developer.webex.com/docs/api/guides/device-xapi) on the far-end device. 
- **`public/`** — Static web UI served at `/`: press-and-hold directional controls and sliders for pan/tilt/zoom ramp speeds. The page expects `?deviceId=<Webex device id>` (same id the service uses for xAPI).
- **`img/`** — Static assets for the UI.

## How it works

1. **Backend (this service)**  
   When your flow POSTs to **`/call`** with a room device serial and a Webex participant callback id, the service caches the device’s Webex **`deviceId`**, sends the participant a direct message (“Camera Controller”), and creates or replaces a **Camera Control** room tab whose URL is **`WEBAPP_PUBLIC_URL/?deviceId=…`**. Opening that tab loads the embedded browser UI.

2. **Far-end camera control**  
   The web UI calls **`POST /api/fecc/command`** with **`action: "rampStart"`** / **`"rampStop"`**, a direction (`up`, `down`, `left`, `right`, `zoom-in`, `zoom-out`), and optional ramp speed fields. The service invokes **`Camera.Ramp`** on the device via Webex xAPI.

3. **Cleanup**  
   **`POST /startup`** and **`POST /call-end`** (with `deviceSerial`) purge bot-authored direct messages for the last known participant **and** remove the Camera Control tab when the service still knows the room id (in-memory cache). Your integration should call these at appropriate lifecycle points.

## Setup

### Setup (bot)

1. [Create a bot](https://developer.webex.com/my-apps/new/bot)
2. Save the bot’s access token for **`WEBEX_BOT_TOKEN`** (see below).
3. In Control Hub, open the **workspace** that contains the room device (not the device detail page alone).
   - In the **Devices** panel, open the menu → **Edit API access**.
   - **Add user or bot**, select your bot, and grant **Full access** so the bot can use the Devices API and xAPI for that workspace’s devices.

### Setup (environment)

1. Copy **`.env.example`** to **`.env`**.
2. Set **`WEBEX_BOT_TOKEN`** to the bot token.
3. Set **`WEBAPP_PUBLIC_URL`** to the **HTTPS** origin where this app is reachable (no trailing slash). Webex loads the room tab in a client web view; the URL must match what you deploy (e.g. reverse proxy or tunnel).
4. Optionally set **`PORT`** (defaults to `3000`).


### Setup (device macro)
In the device macro, replace the example baseUrl (line 3, between quotation marks) with the same value you used in `WEBAPP_PUBLIC_URL` in the **Setup (environment)** section.  You may need to enable macros on the device, but once you have, you can deploy the edited integrated-fecc.js file to the RoomOS device whose camera you want to remotely control.  Make sure to toggle the macro on.

## Run

### Install and start locally

```bash
npm install
npm start
```
The app listens on **`PORT`** and serves the UI from **`/`**. 

## License

All contents are licensed under the MIT license. Please see [license](LICENSE) for details.

## Disclaimer

<!-- Keep the following here -->  
Everything included is for demo and Proof of Concept purposes only. Use of the site is solely at your own risk. This site may contain links to third party content, which we do not warrant, endorse, or assume liability for. These demos are for Cisco Webex usecases, but are not Official Cisco Webex Branded demos.

## Support

Please contact the Webex SD team at [wxsd@external.cisco.com](mailto:wxsd@external.cisco.com?subject=WebexIntegratedFECC) for questions. Or for Cisco internal, reach out to us on Webex App via our bot globalexpert@webex.bot & choose "Engagement Type: API/SDK Proof of Concept Integration Development". 
