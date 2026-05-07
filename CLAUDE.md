# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

K4 Event Viewer — a single-file HTML/JavaScript web app for viewing and filtering vjoon K4 Server `k4_event.log` files. Hosted as a static GitHub Pages site (no backend, no build step).

## Architecture

- **Single-file app**: All HTML, CSS, and JavaScript in one `.html` file (or minimal files). No bundler, no framework, no Node.js.
- **Client-side only**: Users drag/drop or upload `k4_event*.log` files from K4 diagnostics packages. Parsing and display happen entirely in the browser.
- **AWS SDK**: If AWS SDK is needed, use v2 via CDN script tag — not v3 (v3 requires a bundler).

## Key Functionality

- Parse `k4_event.log` files following the format used by K4 Server's `K4_Event_Log_Config.logback`
- Display parsed data as a table with show/hide/reorder columns
- Filter by date, time range, end user, client type
- Export filtered/customized view as Excel-ready CSV

## Deployment

Static GitHub Pages site at `scottdunnflux.github.io/k4-event-viewer/`. No build or deploy commands — push to the appropriate branch and Pages serves it.
