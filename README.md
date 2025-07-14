# n8n with Worker, PostgreSQL, Redis, Browserless and Editly

⚠️ **SECURITY WARNING** ⚠️
- This setup is FOR LOCAL DEVELOPMENT ONLY
- Containers have full host access via mounted Docker socket (/var/run/docker.sock)
- NEVER run this configuration in production environments
- No authentication barriers - all services exposed directly

Starts n8n with:
- PostgreSQL as database
- Redis for queue management
- Browserless (headless Chrome) for web automation
- Editly for video editing
- Separate worker container for offloading executions

## Start

To start all services:
```bash
docker-compose up -d
```

To stop:
```bash
docker-compose stop
```

## Configuration

**IMPORTANT:** Change all default credentials in the [`.env`](.env) file before starting!

### Core Services
- PostgreSQL database credentials in `.env`:
  - POSTGRES_DB
  - POSTGRES_USER
  - POSTGRES_PASSWORD
  - POSTGRES_NON_ROOT_USER
  - POSTGRES_NON_ROOT_PASSWORD

- n8n configuration:
  - ENCRYPTION_KEY (required for credential encryption)
  - N8N_LICENSE_ACTIVATION_KEY (optional)
  - Executions stored for 24 hours (EXECUTIONS_DATA_MAX_AGE)

### Additional Services
- Browserless:
  - Runs on port 1000
  - Access token set via TOKEN environment variable
  - Change default token (currently 'sapidelman')

- Editly:
  - Mounts ./data directory for video file access
  - Uses custom Dockerfile with FFmpeg

- Redis:
  - Used for queue management
  - Data persisted in redis_storage volume

## Usage Examples

### Using Editly via Docker

1. For JSON files with relative paths:
```bash
docker exec editly sh -c "cd /var/data/editly/examples/ && editly audio1.json5 --out /var/data/audio1.mp4"
```

2. For JSON files with absolute paths:
```bash
docker exec editly sh -c "editly /var/data/editly/examples/audio1.json5 --out /var/data/audio1.mp4"
```

Note: Output files will be saved to the mounted ./data directory on host

3. Using Editly from n8n Execute Command node:
```json
{
  "command": "docker exec editly sh -c \"cd /var/data/editly/examples/ && editly audio1.json5 --out /var/data/audio1.mp4\"",
  "parameters": {}
}
```
![image](https://github.com/user-attachments/assets/2d6a807e-0a88-4a43-ab9e-3b77b2cab5b5)


## Security Considerations
- Docker socket mounting gives containers full host control
- All services run as root user
- No network isolation between containers
- Browserless token should be changed from default
- Encryption key should be strong and kept secret
