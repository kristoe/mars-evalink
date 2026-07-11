# Database interface and message flow

This document summarizes how the application connects to PostgreSQL and how station updates flow through the codebase.

## Database connection

The Django application is configured to use PostgreSQL in [evalink/evalink/settings.py](../evalink/evalink/settings.py). The connection uses environment variables for the host, database name, user, password, and port.

In the Docker-based deployment, the web container is configured to reach the database service using:

- Host: `db`
- Port: `5432`
- Database name: `evalink`
- User: `evalink`

These values are defined in [docker-compose.yml](../docker-compose.yml).

## Where the database schema is defined

The database schema is defined with Django ORM models in [evalink/evalink/models.py](../evalink/evalink/models.py). Key models include:

- `Station`
- `PositionLog`
- `TelemetryLog`
- `TextLog`
- `StationMeasure`
- `Campus`
- `Aircraft`
- `AircraftPositionLog`

The migrations in [evalink/evalink/migrations](../evalink/evalink/migrations) apply and evolve that schema.

## How the application interfaces with the database

The application uses Django’s ORM rather than writing raw SQL. The main interaction points are:

- Settings and database configuration: [evalink/evalink/settings.py](../evalink/evalink/settings.py)
- Model definitions and schema: [evalink/evalink/models.py](../evalink/evalink/models.py)
- Runtime writes from incoming messages: [evalink/evalink/handler.py](../evalink/evalink/handler.py)
- Runtime reads for web/API responses: [evalink/evalink/views.py](../evalink/evalink/views.py)

## Station position update flow

A station position update follows this path:

1. An MQTT message arrives in [evalink/evalink/mqtt.py](../evalink/evalink/mqtt.py).
2. The MQTT callback parses the JSON payload and calls `handler.process_message(message)`.
3. In [evalink/evalink/handler.py](../evalink/evalink/handler.py), the handler identifies the station by `hardware_number`.
4. For a `position` message, it creates a new `PositionLog` record with latitude, longitude, altitude, speed, heading, timestamp, and update metadata.
5. The handler saves the new position log and updates the parent `Station` object, including `last_position` and `updated_at`.
6. The handler also creates a `StationMeasure` record containing a snapshot of the station’s feature state.

This typically results in:

- one new row in `PositionLog`
- one update to the `Station` row
- one new row in `StationMeasure`

## Telemetry update flow

A telemetry update follows the same ingress path:

1. The MQTT message is received in [evalink/evalink/mqtt.py](../evalink/evalink/mqtt.py).
2. The message is passed to `handler.process_message(message)`.
3. In [evalink/evalink/handler.py](../evalink/evalink/handler.py), the handler enters the `telemetry` branch.
4. It creates a `TelemetryLog` row linked to the station and, when available, the current `PositionLog`.
5. It stores telemetry values such as temperature, humidity, pressure, wind, battery, voltage, and current.
6. The station’s JSON `features` field is updated and the station is saved again.
7. A `StationMeasure` row is created as a snapshot of the updated state.

This typically results in:

- one new row in `TelemetryLog`
- one update to the `Station` row
- one new row in `StationMeasure`

## Text-message flow

A text-message update follows the same pattern:

1. The MQTT message is received in [evalink/evalink/mqtt.py](../evalink/evalink/mqtt.py).
2. The message is passed to `handler.process_message(message)`.
3. In [evalink/evalink/handler.py](../evalink/evalink/handler.py), the handler enters the `text` branch.
4. It creates a `TextLog` row linked to the station and current position context.
5. The message text is stored, along with a serial number and update timestamps.
6. The station’s feature JSON is updated to include the message in the `texts` list.
7. The station is saved and a `StationMeasure` row is created.

This typically results in:

- one new row in `TextLog`
- one update to the `Station` row
- one new row in `StationMeasure`

## Read-side usage

The web UI and JSON endpoints query the database through Django ORM calls in [evalink/evalink/views.py](../evalink/evalink/views.py). Those views use lookups such as `.get()`, `.filter()`, and `.order_by()` to build the map and reporting data shown to users.
