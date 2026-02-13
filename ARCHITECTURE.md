<!-- AUTO-DOC: Update me when project structure or architecture changes -->

# Architecture

AllBot is a multi-protocol bot with a FastAPI admin server and a plugin/adapter ecosystem.
Runtime updates are handled by the admin update subsystem (download -> backup -> apply).
The adapter layer now includes QQ, Telegram, Web, Win, and wx-filehelper-api bridges.
Adapters are started inside bot core initialization before message listening starts.

- `adapter/INDEX.md`: Multi-platform adapters and queue bridge contracts
- `admin/INDEX.md`: Admin server, update pipeline, and related APIs
