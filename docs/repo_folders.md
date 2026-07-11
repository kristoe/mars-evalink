# Repository Folder Overview

This document summarizes the main folders in the Mars Evalink repository and the role of each area.

## Top-level structure

- **docs/** — Project documentation, architecture notes, and flow diagrams.
- **evalink/** — Main Django application and backend code.
- **mosquitto/** — MQTT broker configuration and startup scripts.
- **scratch/** — Temporary or exploratory files.
- **speech/** — Speech-related utilities and audio experiments.

## Folder details

### docs/
Purpose: Documentation and reference material for the system.

Contains:
- Architecture and system-overview notes
- Interface and data-flow documentation
- Design explanations for the UI, MQTT integration, and database flow

### evalink/
Purpose: The core application package for the Django project.

Contains:
- **evalink/** — Django project package with settings, URL configuration, views, forms, models, and handlers.
- **migrations/** — Database schema migration history for the application.
- **management/commands/** — Custom Django management commands.
- **static/** — Static web assets such as CSS, JavaScript, or images.
- **templates/** — HTML templates for the web UI.
- **tests/** and test modules — Automated tests for application behavior.

### evalink/evalink/
Purpose: The main Django project package.

Contains:
- Application logic for models, admin, MQTT handling, export utilities, and user-facing views
- Configuration for Django settings and runtime entrypoints
- Core modules that connect the UI, database, and messaging systems

### evalink/migrations/
Purpose: Tracks schema changes for the database over time.

Contains:
- One migration file per database change
- Historical versioning for models and related fields

### evalink/management/commands/
Purpose: Holds custom command-line utilities for Django management.

These are typically used for maintenance, data import/export, or operational tasks.

### evalink/static/
Purpose: Stores static frontend resources.

Used for site assets that are served directly by Django.

### evalink/templates/
Purpose: Stores HTML templates used by the web application.

Used for rendering pages such as dashboards, reports, and interactive interfaces.

### mosquitto/
Purpose: Contains configuration and startup files for the Mosquitto MQTT broker.

Contains:
- Broker configuration
- Container entrypoint logic
- Local MQTT server setup for the application

### scratch/
Purpose: A place for temporary, experimental, or in-progress work.

This folder is useful for throwaway scripts or notes that are not yet part of the main codebase.

### speech/
Purpose: Holds speech-processing and audio-related utilities.

Contains:
- Scripts for reading messages over TCP
- Microphone/audio testing utilities
- Related notes and documentation

## Notes

The repository is organized around a Django backend with supporting infrastructure for MQTT messaging, documentation, and speech-related tooling. The folders above reflect the main functional areas of the project.
