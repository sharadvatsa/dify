# Enhanced Dify Sandbox - Dynamic Library Management

## Overview

This enhanced version of the Dify Sandbox now supports dynamic installation and management of both Python and Node.js libraries. The system automatically installs packages from configuration files when the sandbox starts and periodically updates them.

## Features

### 1. Dynamic Python Package Management
- **Configuration**: Add Python packages to `dependencies/python-requirements.txt`
- **Format**: Standard pip requirements format (`package==version` or `package`)
- **Auto-installation**: Packages are installed on startup and periodically refreshed
- **API Endpoints**: 
  - `GET /dependencies?language=python3` - List installed Python packages
  - `POST /dependencies/refresh?language=python3` - Refresh Python packages

### 2. Dynamic Node.js Package Management (NEW)
- **Configuration**: Add Node.js packages to `dependencies/node-requirements.txt`
- **Format**: npm package format (`package@version` or `package`)
- **Auto-installation**: Packages are installed on startup and periodically refreshed
- **API Endpoints**:
  - `GET /dependencies?language=nodejs` - List installed Node.js packages
  - `POST /dependencies/refresh?language=nodejs` - Refresh Node.js packages

### 3. Enhanced Configuration Options

#### Docker Volumes Configuration (`docker/volumes/sandbox/conf/config.yaml`)
```yaml
app:
  port: 8194
  debug: True
  key: dify-sandbox
max_workers: 4
max_requests: 50
worker_timeout: 5
python_path: /usr/local/bin/python3
python_lib_path:
  - /usr/local/lib/python3.10
  - /usr/lib/python3.10
  - /usr/lib/python3
  - /usr/lib/x86_64-linux-gnu
  # ... other paths
nodejs_path: /usr/local/bin/node
nodejs_lib_path:
  - /usr/local/lib/node_modules
  - /usr/lib/node_modules
  - /var/sandbox/sandbox-nodejs/nodejs-project/node_temp/node_modules
npm_mirror_url: ''  # Optional: specify npm registry mirror
nodejs_deps_update_interval: 30m  # How often to update Node.js dependencies
enable_network: True
```

#### Environment Variables Support
- `NODEJS_LIB_PATH` - Comma-separated list of Node.js library paths
- `NPM_MIRROR_URL` - npm registry mirror URL
- `NODEJS_DEPS_UPDATE_INTERVAL` - Update interval for Node.js dependencies

## Usage Examples

### Adding Python Packages
Edit `dependencies/python-requirements.txt`:
```
requests==2.31.0
pandas==2.0.3
numpy==1.24.3
matplotlib==3.7.1
beautifulsoup4
```

### Adding Node.js Packages
Edit `dependencies/node-requirements.txt`:
```
lodash@4.17.21
axios@1.6.0
moment@2.29.4
@types/node@18.0.0
uuid
```

### Using Packages in Code

#### Python Example
```python
import requests
import pandas as pd
import numpy as np

# Your code here using the installed packages
response = requests.get('https://api.example.com/data')
df = pd.DataFrame(response.json())
```

#### Node.js Example
```javascript
const _ = require('lodash');
const axios = require('axios');
const moment = require('moment');

// Your code here using the installed packages
const data = _.groupBy(someArray, 'category');
const response = await axios.get('https://api.example.com/data');
const now = moment().format('YYYY-MM-DD');
```

## API Reference

### List Dependencies
```bash
# Python dependencies
curl -X GET "http://localhost:8194/dependencies?language=python3" \
  -H "Authorization: Bearer dify-sandbox"

# Node.js dependencies
curl -X GET "http://localhost:8194/dependencies?language=nodejs" \
  -H "Authorization: Bearer dify-sandbox"
```

### Refresh Dependencies
```bash
# Python dependencies
curl -X POST "http://localhost:8194/dependencies/refresh?language=python3" \
  -H "Authorization: Bearer dify-sandbox"

# Node.js dependencies
curl -X POST "http://localhost:8194/dependencies/refresh?language=nodejs" \
  -H "Authorization: Bearer dify-sandbox"
```

### Run Code with Dependencies
```bash
# Python
curl -X POST "http://localhost:8194/run" \
  -H "Authorization: Bearer dify-sandbox" \
  -H "Content-Type: application/json" \
  -d '{
    "language": "python3",
    "code": "import requests; print(requests.get(\"https://httpbin.org/json\").json())",
    "enable_network": true
  }'

# Node.js
curl -X POST "http://localhost:8194/run" \
  -H "Authorization: Bearer dify-sandbox" \
  -H "Content-Type: application/json" \
  -d '{
    "language": "nodejs",
    "code": "const _ = require(\"lodash\"); console.log(_.version);",
    "enable_network": false
  }'
```

## Installation and Deployment

1. **Update Requirements Files**: Add your required packages to the respective requirements files
2. **Rebuild Sandbox**: The Docker container will automatically install the packages on startup
3. **Monitor Logs**: Check the container logs to ensure packages are installed successfully

## Security Considerations

- Network access is controlled by the `enable_network` flag in both configuration and runtime
- Package installation respects the configured proxy settings
- Periodic updates ensure packages stay current but can be disabled by setting update intervals to `0s`
- All package installations are logged for audit purposes

## Troubleshooting

### Common Issues

1. **Package Installation Fails**
   - Check network connectivity
   - Verify package names and versions
   - Check proxy settings if behind a firewall

2. **Package Not Found in Code**
   - Ensure package is listed in the requirements file
   - Check if the sandbox has been restarted after adding the package
   - Use the refresh dependencies API endpoint

3. **Performance Issues**
   - Consider reducing the frequency of dependency updates
   - Monitor resource usage during package installation

### Logs
Check the sandbox container logs for detailed information about package installation:
```bash
docker logs sandbox
```

## Migration from Previous Version

If you're upgrading from a previous version:

1. Your existing Python packages in `python-requirements.txt` will continue to work
2. Add the new Node.js configuration sections to your `config.yaml`
3. Create `node-requirements.txt` for any Node.js packages you need
4. Restart the sandbox container to apply changes
