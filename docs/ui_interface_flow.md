# UI Interface Flow

## Overview

The main user interface is a browser-based operational map dashboard built around Django and OpenLayers. The primary page is rendered from `evalink/evalink/templates/map.html` and interacts with Django views in `evalink/evalink/views.py`.

## Main UI Elements

### 1. Map canvas
- The central interface is the interactive OpenLayers map.
- It displays stations, tracks, aircraft, and the selected base-map layer.

### 2. Upper-left coordinate panel
- A small floating panel in the upper-left corner shows the current mouse cursor latitude and longitude.
- It is rendered by the `geo-tab` container and updated from the `geo-coordinates` element.
- The coordinates are refreshed during pointer movement events on the map.

### 3. Far-right marker/station panel
- A floating panel on the right side lists stations or markers.
- It is implemented by the `marker-list` container.
- The list is populated from the current GeoJSON feature data and clicking an entry triggers path loading and map zooming.

### 4. Bottom-right track/crew panel
- A floating panel in the lower-right area contains the track timeline controls.
- It includes:
  - the time slider,
  - the crew dropdown,
  - buttons to add, remove, and view crew assignments,
  - and track-history navigation.
- It is implemented by the `time-slider-container` container.

### 5. Left-side search and chat panel
- A collapsible drawer on the left contains:
  - campus selection,
  - date-range search fields,
  - history search results,
  - and the chat/message list UI.
- It is implemented by the `search-panel` container.
- It includes the `campusDropdown`, `date-input`, `endDate`, `historyDropdown`, `message-list`, and the chat form.

### 6. Emergency panel
- There is also an emergency-device panel that can appear for emergency-related device tracking.
- It is implemented by the `emergency-device-panel` container.

## How the UI is Event Driven

The interface is driven by browser-side events and periodic refreshes.

### Browser events
- Map clicks and taps trigger station selection, path loading, and popup behavior.
- Pointer movement updates the coordinate display and hover information.
- Keyboard events are used for shortcuts such as map rotation reset.
- Form submissions and button clicks trigger actions such as sending chat messages or saving planner locations.

### Periodic updates
- The map refreshes GeoJSON station data on a timer.
- Aircraft data is refreshed on a separate timer.
- Message text is periodically polled so the UI stays current.

## How the UI Interfaces with the Underlying Logic

The UI is not self-contained. It talks to the backend through Django endpoints.

### Data requests
- The map requests station/feature data from `features.json`.
- Track history comes from `path.json`.
- Messages come from `texts.json`.
- Aircraft data comes from `aircraft.json`.
- Campus data comes from `campuses.json`.
- Chat and planning actions are sent to Django endpoints such as `chat/`, `add-location-to-plan`, and `delete-planner-point`.

### Backend responsibilities
- Django views in `evalink/evalink/views.py` query the database using the Django ORM.
- MQTT messages are handled separately by the ingestion pipeline in `evalink/evalink/mqtt.py` and `evalink/evalink/handler.py`.
- The database stores stations, positions, telemetry, messages, and other operational data.

## Layering and Visual Elements

The map uses multiple layers:
- a base-map tile layer,
- a vector layer for stations/features,
- a vector track layer for paths,
- and an aircraft layer.

The visible icons and symbols are mostly rendered directly in the browser using OpenLayers and SVG-based icon generation. They are not currently configured through a dedicated visual component library or admin interface.

## Summary

The UI is a layered, map-first dashboard with floating panels for coordinates, stations, track history, crew actions, and search/chat functions. It is event-driven in the browser, refreshed from the backend, and connected to the underlying Django application through structured HTTP endpoints and the ORM.
