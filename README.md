# Ollama Home Server

Production-ready Ollama server setup for serving AI models to other services in your home network.

## Quick Start

```bash
# Start the server
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f ollama

# Stop the server
docker-compose down
```

### GPU Support

To enable GPU acceleration:

1. **Update .env file:**
   ```bash
   USE_GPU=true
   OLLAMA_NUM_GPU=1
   ```

2. **Enable GPU in docker-compose.yml:**
   Uncomment the GPU reservation section:
   ```yaml
   reservations:
     devices:
       - capabilities: [ gpu ]
   ```

3. **Ensure nvidia-docker is installed** on your host system

## Configuration

The server is configured via [.env](.env.example) file:

| Variable | Default | Description |
|----------|---------|-------------|
| `OLLAMA_PORT` | 11435 | External port (avoids conflict with other instances) |
| `OLLAMA_NUM_PARALLEL` | 1 | Number of concurrent requests |
| `OLLAMA_MAX_LOADED_MODELS` | 1 | Models kept in memory |
| `OLLAMA_KEEP_ALIVE` | 5m | How long models stay loaded |
| `USE_GPU` | false | Enable GPU passthrough (requires uncommenting GPU config) |
| `OLLAMA_NUM_GPU` | 0 | Number of GPUs to use (0 for CPU only) |
| `MEMORY_LIMIT` | 8g | Maximum memory usage |
| `CPU_LIMIT` | 4 | Maximum CPU cores |

## API Access

- **Local**: `http://localhost:11435`
- **LAN**: `http://<server-ip>:11435`
- **Health Check**: `curl http://localhost:11435/api/tags`

## Management Commands

```bash
# Pull a model
docker-compose exec ollama ollama pull llama3.2

# List installed models
docker-compose exec ollama ollama list

# Remove a model
docker-compose exec ollama ollama rm llama3.2

# Check container stats
docker stats ollama-server

# View detailed logs
docker-compose logs ollama --since 1h

# Restart service
docker-compose restart ollama
```

## Data Persistence

Models and configuration are stored in the `ollama-data` Docker volume:

```bash
# Backup models
docker run --rm -v ollama-data:/data -v $(pwd):/backup alpine tar czf /backup/ollama-backup.tar.gz -C /data .

# Restore models
docker run --rm -v ollama-data:/data -v $(pwd):/backup alpine tar xzf /backup/ollama-backup.tar.gz -C /data
```

## Security Considerations

- **Network Access**: Port 11435 is exposed to LAN only
- **Firewall**: Ensure proper firewall rules for your network
- **Updates**: Keep Docker images updated with `docker-compose pull`
- **Monitoring**: Health checks run every 30 seconds

## Troubleshooting

### Service Won't Start
```bash
# Check logs for errors
docker-compose logs ollama

# Verify GPU access (if using GPU)
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

### High Memory Usage
```bash
# Check current resource usage
docker stats ollama-server

# Adjust limits in .env file
MEMORY_LIMIT=4g
OLLAMA_MAX_LOADED_MODELS=1
```

### Connection Issues
```bash
# Test API connectivity
curl -v http://localhost:11435/api/tags

# Check port binding
docker-compose port ollama 11434
```

## Integration Examples

### Python Client
```python
import requests

response = requests.get('http://server-ip:11435/api/tags')
models = response.json()
```

### Docker Compose Service
```yaml
services:
  my-app:
    image: my-app:latest
    environment:
      - OLLAMA_URL=http://ollama-server:11434
    depends_on:
      - ollama-server
```

## Monitoring

The service includes built-in health checks and structured logging:

- **Health Status**: `docker-compose ps`
- **Resource Usage**: `docker stats ollama-server`
- **Logs**: `docker-compose logs ollama`
- **API Status**: `curl http://localhost:11435/api/tags`