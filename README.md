# Proxy Cluster Documentation

This repository contains a proxy cluster configuration using OpenResty (Nginx) that forwards requests to different backend services based on the host header.

## Architecture

The proxy cluster consists of:
- **Main proxy server** listening on ports 9080, 9081, and 9082
- **Multiple backend services** mapped through configuration files
- **Health check endpoint** available at `/status`

## Configuration Structure

### Main Configuration
- `conf/nginx.conf` - Main Nginx configuration with performance optimizations
- `conf/conf.d/stands.conf` - Proxy server blocks for different ports

### Mapping Files
- `conf/mappings/stand_9080.map` - Mapping for Stand A services (port 9081)
- `conf/mappings/stand_9081.map` - Mapping for Stand B services (port 9082)

### MIME Types
- `conf/mime.types.minimal` - Optimized minimal MIME types file (1 line)

## Ports and Services

### Port 9080
- **Purpose**: Default proxy endpoint
- **Function**: Serves health check page and forwards requests
- **Backend Mapping**: See `stand_9080.map`

### Port 9081
- **Purpose**: Proxy for Stand A services
- **Function**: Forwards requests to backend services
- **Backend Mapping**: See `stand_9080.map` (mapped to port 9080)

### Port 9082
- **Purpose**: Proxy for Stand B services
- **Function**: Forwards requests to backend services
- **Backend Mapping**: See `stand_9081.map` (mapped to port 9081)

## Mapping Files

### stand_9080.map
```
prod.internal 10.0.1.10:8080;
staging.internal 10.0.1.11:8080;
api.internal 10.0.1.12:8080;
```

### stand_9081.map
```
web.internal 10.0.2.10:8080;
admin.internal 10.0.2.11:8080;
service.internal 10.0.2.12:8080;
```

## Health Check

Access the health check endpoint at:
```
http://localhost:9080/status
```

## Performance Optimizations

The proxy cluster includes several performance optimizations:
- **Minimal MIME types**: `mime.types.minimal` reduces memory usage from 2000+ entries to 1 entry
- **Connection handling**: Optimized worker connections and timeouts
- **Security**: Server tokens disabled, clean headers
- **Resource limits**: Configured body size and timeout limits

## Usage

1. The proxy forwards requests based on the `Host` header
2. All backend services receive the original request with all headers preserved
3. No additional MIME type headers are added by the proxy
4. Only the HTML info page is served directly by Nginx

## Security Considerations

- **Header Preservation**: All original headers are passed through to backend services
- **No Content-Type Modification**: Proxy does not add or modify MIME type headers
- **Access Control**: Health check endpoint only accessible from localhost
- **Security Headers**: Server tokens removed for security

## Directory Structure

```
.
├── conf/
│   ├── mime.types.minimal          # Optimized MIME types file (1 line)
│   ├── nginx.conf                  # Main Nginx configuration with optimizations
│   ├── conf.d/
│   │   └── stands.conf             # Proxy server blocks for ports 9080-9082
│   └── mappings/
│       ├── stand_9080.map          # Port 9081 mappings (Stand A)
│       └── stand_9081.map          # Port 9082 mappings (Stand B)
├── html/                           # HTML files
│   └── index.html                  # Health check page with updated content
└── README.md                       # This file
```

## Deployment Instructions

### Prerequisites
- Docker and Docker Compose installed
- Proper file permissions for Nginx to read configuration files

### Deployment Steps
1. **Set proper file permissions**:
   ```bash
   sudo chmod 644 conf/mime.types.minimal
   sudo chmod 644 conf/nginx.conf
   sudo chmod 644 conf/conf.d/stands.conf
   sudo chmod 644 conf/mappings/stand_9080.map
   sudo chmod 644 conf/mappings/stand_9081.map
   sudo chmod 644 html/index.html
   ```

2. **Start the proxy cluster**:
   ```bash
   docker-compose up -d
   ```

3. **Verify deployment**:
   ```bash
   docker-compose ps
   curl http://localhost:9080/
   curl http://localhost:9080/status
   ```

### File Permissions Explanation
- **644 permissions**: Readable by all, writable only by owner
- **Configuration files**: Must be readable by Nginx process
- **HTML files**: Must be readable by Nginx for serving
- **Security**: Prevents unauthorized modification while allowing normal operation

### Testing
After deployment, verify:
- HTML info page is accessible at `http://localhost:9080/`
- Health check endpoint works at `http://localhost:9080/status`
- Proxy forwarding works correctly to backend services
- No MIME type header modifications occur
