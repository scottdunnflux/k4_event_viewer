# K4 Event Viewer

A browser-based tool for viewing, filtering, and analyzing vjoon K4 Server `k4_event.log` files. Built for K4 administrators and power users who need to make sense of event log data without command-line tools or log parsers.

All processing happens in your browser. No data is uploaded to any server.

## Usage

The app is live at **[https://scottdunnflux.github.io/k4_event_viewer/](https://scottdunnflux.github.io/k4_event_viewer/)**

1. Download the K4 diagnostics package from the **K4 Server** section of K4 Admin.
2. Extract the package and locate the `k4_event*.log` files in the K4 Server Logs folder.
3. Drag and drop the log file(s) onto the page (or click Browse).

The interface should be self-explanatory. Click the **Help** button in the app for details on fields, features, and tips.

### Offline Use

Download `index.html` and open it directly in any modern browser. No install, no dependencies, no internet required.

## Enabling Event Logging

Event logging is disabled by default on K4 Server. To enable it, edit `K4_Event_Log_Config.logback` at `C:\ProgramData\vjoon\K4_Server` and set the logger level to `INFO` for `com.vjoon.VJ_AuditEvent`.

## Features

- Parse and merge multiple `k4_event*.log` files
- Filter by date, time, user, client type, object type, action, publication, session, or free text
- Sort by any column (shift+click for multi-column sort)
- Resizable columns with show/hide toggles
- Text object grouping — collapses child objects under parent articles, reducing noise by ~79%
- Clickable session IDs to isolate a session
- Smart data parsing — click any data cell to see structured mail, login, XML, or comment details
- Custom article labels that persist across sessions
- Local timezone conversion
- CSV export (flat, summary, or grouped detail modes)

## License

MIT License

Copyright (c) 2026 Scott Dunn

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

## Credits

Built by Scott Dunn ([Flux Consulting](https://fluxconsulting.com)) with Claude (Anthropic).
