# K4 Event Viewer

K4 Server tracks user activity in a k4_event.log. This log is disabled by default. Enable it in `K4_Event_Log_Config.logback` located at `C:\ProgramData\vjoon\K4_Server`
The log can be challenging to work with. This tool aims to make it eaiser to view and filter K4's k4_event.log files.

# Approach
In this case we expect K4 administrators/powerusers to download the k4 diagnostics package from the `K4 Server` section of K4 Admin. Then expand that package, locate the k4_event*.log files and drag them to this project's web page. The web page is a single file page html/javascript. It accepts uploaded k4_event*.log files and parses them following guidelines specified in ../enrich-k4_event-logs/ . That project uses bash to parse the logs. In this project, javascript will be used.
Once parsed, the data will be displayed as a table.
Interface will allow columns to be hidden or shown or re-ordered. 
Data should be filterable. Prioritize dates, time ranges, end users, client type
There should be a way to export the filtered/customized view as an excel ready csv file.

# Use
Users find the web page at the URL for this repo (likely scottdunnflux.github.io/k4-event-viewer/ )
Upload k4_event.log files and make selections.

