# Uptime Kuma REST API Wrapper

A comprehensive REST API wrapper for Uptime Kuma's Socket.io API, enabling full monitor, notification, and status page management via simple HTTP endpoints and curl commands.

**✅ Compatible with Uptime Kuma 2.3.2**

## Features

- **23 REST API Endpoints** for complete Uptime Kuma management
- **Monitor Operations**: Create, list, update, pause, resume, delete
- **Bulk Operations**: Update, control, and manage multiple monitors at once
- **Notification Management**: Create, list, test, delete notifications
- **Status Page Management**: Create, read, update, delete status pages and assign monitors
- **Advanced Filtering**: By group, tag, name patterns (wildcard), and monitor type
- **Bulk Notification Assignment**: Set, add, or remove notifications from multiple monitors
- **Query Parameter Filtering**: Use simple `?group=X&tag=Y` instead of JSON for all bulk operations
- **.env Configuration**: Simple environment variable setup
- **Zero External Dependencies**: Just curl for all operations

## Quick Start

### 1. Installation

```bash
git clone https://github.com/MASHINISTPROJECT/uptime-kuma-rest-api.git
cd uptime-kuma-rest-api
pip install -r requirements.txt
```

### 2. Configuration

Copy the example environment file and configure it:

```bash
cp .env.example .env
```

Edit `.env`:
```bash
UPTIME_KUMA_URL=http://localhost:3001
UPTIME_KUMA_USERNAME=admin
UPTIME_KUMA_PASSWORD=your_password_here
```

### 3. Start the API

```bash
python uptime_kuma_rest_api.py
```

The API will be available at `http://127.0.0.1:5001`

## API Endpoints

### System
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check (connection + auth status) |
| POST | `/connect` | Manually reconnect to Uptime Kuma |

### Monitors
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/monitors` | List all monitors (supports filtering) |
| POST | `/monitors` | Create a single monitor |
| POST | `/monitors/bulk` | Create multiple monitors at once |
| PUT | `/monitors/bulk-update` | Bulk update monitors by filters |
| DELETE | `/monitors/<id>` | Delete a monitor |
| POST | `/monitors/<id>/pause` | Pause a monitor |
| POST | `/monitors/<id>/resume` | Resume a monitor |
| POST | `/monitors/bulk-control` | Bulk pause/resume/delete by filters |

### Notifications
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/notifications` | List all notifications |
| POST | `/notifications` | Create a notification |
| DELETE | `/notifications/<id>` | Delete a notification |
| POST | `/notifications/<id>/test` | Test a notification |

### Notification Assignment
| Method | Endpoint | Description |
|--------|----------|-------------|
| PUT | `/monitors/bulk-notifications` | Add/remove notifications to monitors |
| PUT | `/monitors/set-notifications` | Replace all notifications on monitors |

### Status Pages
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/status-pages` | List all status pages |
| POST | `/status-pages` | Create a new status page |
| GET | `/status-pages/<slug>` | Get a status page by slug |
| PUT | `/status-pages/<slug>` | Update a status page |
| DELETE | `/status-pages/<slug>` | Delete a status page |
| POST | `/status-pages/<slug>/monitors` | Add monitors to a status page |

### Settings
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/settings` | Get Uptime Kuma settings |

## Filtering Options

All bulk operations and monitor listing support these filters:

| Parameter | Example | Description |
|-----------|---------|-------------|
| `group` | `?group=Production` | Filter by monitor group name |
| `tag` | `?tag=critical` | Filter by tag name |
| `name_pattern` | `?name_pattern=web-*` | Filter by name with wildcards (`*`, `?`) |
| `type` | `?type=http` | Filter by monitor type (`http`, `ping`, `dns`, etc.) |
| `include_groups` | `?include_groups=true` | Include group monitors in results |

Filters use **AND** logic when combined. Query parameters take precedence over JSON body filters.

## Usage Examples

### Health Check

```bash
curl http://127.0.0.1:5001/health
```

### List Monitors with Filters

```bash
# All monitors
curl http://127.0.0.1:5001/monitors

# By type
curl "http://127.0.0.1:5001/monitors?type=ping"

# By name pattern
curl "http://127.0.0.1:5001/monitors?name_pattern=*Bulk*"

# By group
curl "http://127.0.0.1:5001/monitors?group=Production"
```

### Create Monitors

```bash
# Single monitor
curl -X POST http://127.0.0.1:5001/monitors \
  -H "Content-Type: application/json" \
  -d '{"name":"My Website","url":"https://example.com","type":"http"}'

# Bulk creation
curl -X POST http://127.0.0.1:5001/monitors/bulk \
  -H "Content-Type: application/json" \
  -d '[{"name":"Site 1","url":"https://site1.com"},{"name":"Site 2","url":"https://site2.com"}]'
```

### Bulk Update Monitors

```bash
# Update all monitors with tag "production"
curl -X PUT "http://127.0.0.1:5001/monitors/bulk-update?tag=production" \
  -H "Content-Type: application/json" \
  -d '{"updates":{"interval":600,"timeout":45}}'

# Update monitors matching name pattern
curl -X PUT "http://127.0.0.1:5001/monitors/bulk-update?name_pattern=*api*" \
  -H "Content-Type: application/json" \
  -d '{"updates":{"maxretries":5}}'
```

### Bulk Monitor Control

```bash
# Pause all ping monitors
curl -X POST "http://127.0.0.1:5001/monitors/bulk-control?type=ping&action=pause"

# Resume monitors by name pattern
curl -X POST "http://127.0.0.1:5001/monitors/bulk-control?name_pattern=*Bulk*&action=resume"

# Delete monitors by tag
curl -X POST "http://127.0.0.1:5001/monitors/bulk-control?tag=test&action=delete"
```

### Single Monitor Control

```bash
curl -X POST http://127.0.0.1:5001/monitors/1/pause
curl -X POST http://127.0.0.1:5001/monitors/1/resume
curl -X DELETE http://127.0.0.1:5001/monitors/1
```

### Notification Management

```bash
# List notifications (simplified - shows IDs and types)
curl "http://127.0.0.1:5001/notifications?simple=true"

# Create a webhook notification
curl -X POST http://127.0.0.1:5001/notifications \
  -H "Content-Type: application/json" \
  -d '{"name":"My Webhook","type":"webhook","webhookURL":"https://hooks.example.com"}'

# Create a Discord notification
curl -X POST http://127.0.0.1:5001/notifications \
  -H "Content-Type: application/json" \
  -d '{"name":"Discord Alerts","type":"discord","discordWebhookUrl":"https://discord.com/api/webhooks/..."}'

# Create an email notification
curl -X POST http://127.0.0.1:5001/notifications \
  -H "Content-Type: application/json" \
  -d '{"name":"Email Alerts","type":"smtp","smtpHost":"smtp.example.com","smtpPort":587,"smtpUsername":"user","smtpPassword":"pass"}'

# Test a notification
curl -X POST http://127.0.0.1:5001/notifications/1/test

# Delete a notification
curl -X DELETE http://127.0.0.1:5001/notifications/1
```

### Set Notifications on Monitors

```bash
# Set notification ID 3 for all monitors
curl -X PUT "http://127.0.0.1:5001/monitors/set-notifications?notification_ids=3" \
  -H "Content-Type: application/json"

# Set notifications for monitors with specific tag
curl -X PUT "http://127.0.0.1:5001/monitors/set-notifications?tag=critical&notification_ids=1,2" \
  -H "Content-Type: application/json"

# Clear all notifications from monitors in a group
curl -X PUT "http://127.0.0.1:5001/monitors/set-notifications?group=Test&notification_ids=" \
  -H "Content-Type: application/json"
```

### Add/Remove Notifications (Advanced)

```bash
# Add notifications to monitors
curl -X PUT "http://127.0.0.1:5001/monitors/bulk-notifications?action=add&notification_ids=1,2" \
  -H "Content-Type: application/json"

# Remove notification from monitors
curl -X PUT "http://127.0.0.1:5001/monitors/bulk-notifications?action=remove&notification_ids=1" \
  -H "Content-Type: application/json"
```

### Status Page Management

```bash
# List all status pages
curl http://127.0.0.1:5001/status-pages

# Get a specific status page
curl http://127.0.0.1:5001/status-pages/my-page

# Create a status page
curl -X POST http://127.0.0.1:5001/status-pages \
  -H "Content-Type: application/json" \
  -d '{"title":"My Status Page","slug":"my-page"}'

# Update a status page
curl -X PUT http://127.0.0.1:5001/status-pages/my-page \
  -H "Content-Type: application/json" \
  -d '{"description":"Updated description"}'

# Add monitors to a status page
curl -X POST http://127.0.0.1:5001/status-pages/my-page/monitors \
  -H "Content-Type: application/json" \
  -d '{"monitor_ids":[1,2,3]}'

# Delete a status page
curl -X DELETE http://127.0.0.1:5001/status-pages/my-page
```

### Get Settings

```bash
curl http://127.0.0.1:5001/settings
```

## Response Format

All endpoints return JSON:

**Success:**
```json
{
  "success": true,
  "monitorID": 1,
  "message": "Monitor created successfully"
}
```

**Bulk operations:**
```json
{
  "results": [...],
  "total": 5,
  "successful": 5,
  "failed": 0
}
```

**Error:**
```json
{
  "success": false,
  "error": "Error description"
}
```

## Configuration

Environment variables in `.env`:

```bash
# Required - Uptime Kuma connection
UPTIME_KUMA_URL=http://localhost:3001
UPTIME_KUMA_USERNAME=admin
UPTIME_KUMA_PASSWORD=your_password

# Optional - API server settings
API_HOST=127.0.0.1
API_PORT=5001
API_DEBUG=false
```

## Requirements

- Python 3.7+
- Uptime Kuma 2.3.2 (tested)
- Network access to Uptime Kuma instance

## Limitations

- Uses Uptime Kuma's internal Socket.io API (not officially supported)
- May break with future Uptime Kuma updates
- Notification test may fail for certain notification types (depends on Uptime Kuma's test support)

## Version Compatibility

| UK Version | Status |
|------------|--------|
| 2.3.2 | ✅ Fully tested (23 endpoints) |
| 2.x | Should work (minor changes possible) |
| 1.x | Not supported |

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) file for details

## Disclaimer

This project uses Uptime Kuma's internal Socket.io API which is not officially supported for third-party integrations. Use at your own risk and expect potential breaking changes with Uptime Kuma updates.