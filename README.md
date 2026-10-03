# Plantagotchi

A React Native / Expo plant-care prototype that combines plant identification, care-data lookup, live sensor-state visualization, and an illustrated virtual-companion interface.

The repository contains three runnable pieces:

1. an Expo mobile/web client
2. a TypeScript WebSocket mock-sensor server
3. a small React/Vite control panel for changing simulated sensor values

> **Hardware scope:** the checked-in code consumes sensor-shaped telemetry but does not contain Arduino/microcontroller firmware. Hardware acquisition is therefore outside the implementation documented here.

## Mobile application

The main client lives in <code>plant-whisperer/</code> and uses Expo Router.

### Plant identification

The camera flow can:

- capture a photo with Expo Camera
- select an image from the gallery
- submit the image to PlantNet
- display the top identification result and alternatives
- fetch care fields from Perenual
- fetch environment ranges from Plantbook
- cache selected species/care data with AsyncStorage
- continue into the dashboard

The manual species-selection screen currently exists as a placeholder and is not yet implemented beyond navigation.

### Live plant state

The client accepts this telemetry shape over WebSocket:

~~~text
soil  - soil-moisture reading
temp  - temperature
hum   - relative humidity
mq2   - gas / air-quality reading
rain  - surface-wetness / watering signal
bio   - biosignal metric
~~~

<code>usePlantState</code> converts those raw values into hydration, comfort, air-quality, and biosignal scores. <code>plantModel.ts</code> then derives plant states and messages from explicit thresholds.

Examples from the current source include:

- temperature: cold below 13 °C, stable from 13–27 °C, hot above 27 °C
- humidity: dry below 35%, normal from 35–80%, humid above 80%
- soil: dry / thirsty / okay / hydrated bands
- air quality: good / bad / polluted bands based on the MQ-2-style reading
- watering detection from the rain/wetness channel
- a moving baseline for the biosignal channel

The dashboard records emotion changes in a short event log and can create a watering reminder when hydration falls below the configured threshold.

## Mock sensor pipeline

<code>mock-server/server.ts</code> runs an Express HTTP server plus a WebSocket server on port 4000.

It:

- exposes <code>GET /health</code>
- keeps the current six-channel sensor state
- broadcasts the state to connected clients
- supports <code>set</code>, <code>start</code>, and <code>stop</code> WebSocket messages
- streams at a one-second interval by default

The companion <code>sensor-mock-ui/</code> app provides controls for changing values and previewing the exact payload received by the mobile client.

## Stack

### Mobile

- Expo 54
- React 19
- React Native 0.81
- TypeScript
- Expo Router
- Expo Camera / Image Picker
- AsyncStorage
- PlantNet API
- Perenual API
- Plantbook API

### Simulation tooling

- Node.js
- Express
- <code>ws</code>
- TypeScript / <code>tsx</code>
- React + Vite for the sensor-control UI

## Repository layout

~~~text
Plantagotchi/
├── plant-whisperer/    # Expo client
│   ├── app/            # routed screens
│   ├── lib/            # PlantNet client
│   └── src/
│       ├── hooks/
│       ├── services/
│       └── screens/
├── mock-server/        # WebSocket telemetry simulator
├── sensor-mock-ui/     # browser controls for simulator
├── Animations/         # illustrated plant states
└── RUN_INSTRUCTIONS.md
~~~

## Run with simulated sensors

Terminal 1:

~~~bash
cd mock-server
npm install
npm run server
~~~

Terminal 2:

~~~bash
cd plant-whisperer
npm install
npm start
~~~

Optional sensor-control UI:

~~~bash
cd sensor-mock-ui
npm install
npm run dev
~~~

The client automatically handles localhost, Android-emulator, and Expo development-host WebSocket addresses. <code>EXPO_PUBLIC_WS_URL</code> can override the WebSocket endpoint.

## API configuration

The source supports these environment variables:

~~~text
EXPO_PUBLIC_PLANTNET_API_KEY
EXPO_PUBLIC_PERENUAL_API_KEY
EXPO_PUBLIC_PLANTBOOK_TOKEN
EXPO_PUBLIC_WS_URL
~~~

Do not commit private credentials to a public repository. Restrict/rotate provider credentials as appropriate for their intended client-side use.

## Demo

[Plantagotchi demo](https://youtube.com/shorts/_Rtnkhy3jHY?si=Nw0nGcZnq8E2Oxly)

## Current limitations

- no microcontroller/Arduino firmware is included
- manual species selection is still a TODO
- sensor-to-emotion logic is rule/threshold based rather than a learned model
- production deployment/networking for a physical sensor source is not provided by the mock server

## Author

Muhammad Zain Asad — [GitHub](https://github.com/muhzain05)
