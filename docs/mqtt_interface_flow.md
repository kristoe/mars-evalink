# MQTT Interface Flow

## Overview
This application interfaces directly with MQTT using the `paho.mqtt.client` library.
It receives MQTT uplink messages, routes them through Django handler code, and persists relevant data into the database.

## 1. MQTT client startup
- File: `evalink/evalink/__init__.py`
  - imports `evalink/evalink/mqtt.py`
  - if `mqtt.client` exists, calls `mqtt.client.loop_start()`
- This means the MQTT listener starts automatically when the Django app imports the package.

## 2. Connection and subscriptions
- File: `evalink/evalink/mqtt.py`
  - `client = mqtt.Client()`
  - optional TLS via `client.tls_set()`
  - credentials via `client.username_pw_set(...)`
  - connect to broker with `client.connect(MQTT_SERVER, MQTT_PORT, 60)`
- `on_connect()` subscribes to:
  - `${MQTT_TOPIC}/+/json/#` for normal mesh JSON messages
  - `${MQTT_TOPIC}/aircraft/+` for aircraft/drone feeds

## 3. Incoming message processing
- `mqtt.py` defines `on_message()`
  - parses `msg.payload` as JSON
  - for aircraft topics, calls `handler.process_aircraft(hex_code, message)`
  - for normal messages, verifies required fields and calls `handler.process_message(message)`

## 4. `process_message()` handles `position`
- File: `evalink/evalink/handler.py`
- For `message['type'] == 'position'`, the flow is:
  1. `number = message['from']`
  2. look up `Station` by `hardware_number=number`
  3. compute timezone and timestamp from campus settings
  4. extract latitude/longitude from payload fields
  5. create a `PositionLog` record with coordinates, altitude, speed, heading, and timestamp
  6. decide whether to save the log based on geofence and return rules
  7. update `station.last_position`
  8. update `station.features` geometry/properties
  9. save the `station`
  10. call `log_measurements()` to create a `StationMeasure`

## 5. Final persistence
- The position message is persisted to Django models:
  - `PositionLog`
  - `Station`
  - `StationMeasure`
- If the station is unknown, the message is ignored.
- Errors in processing are caught in `mqtt.py`, which closes DB connections and sleeps briefly before continuing.

## 6. Publishing from the app
- The app also publishes MQTT from `evalink/evalink/views.py` for chat messages:
  - `client.publish(topic, data)`
  - uses topic `f'{MQTT_TOPIC}/2/json/mqtt/'`
- This is the downward path from the web UI into the mesh/cloud topic stream.

## 7. Environment-based configuration
- Relevant environment variables:
  - `MQTT_SERVER`
  - `MQTT_PORT`
  - `MQTT_TOPIC`
  - `MQTT_USER`
  - `MQTT_PASSWORD`
  - `MQTT_TLS`
- Tests may disable the real MQTT client with `MQTT_ENABLED = False` in `evalink/evalink/test_settings.py`.

## 8. Summary flow for a `position` payload
1. Broker publishes a `position` JSON message on `${MQTT_TOPIC}/.../json/...`
2. `paho.mqtt.client` receives it in `evalink/evalink/mqtt.py`
3. `on_message()` parses the payload and routes it to `handler.process_message()`
4. `handler.process_message()` handles `type == 'position'`
5. The app saves the message into Django database records

This makes MQTT a direct part of the app’s input pipeline rather than just a secondary integration.
