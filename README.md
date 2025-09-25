# UBF - Universal Backend-to-Frontend Connector Tool

🚀 **UBF** is a powerful CLI tool that automatically generates frontend SDKs (functions, hooks, services) from any backend API. It supports OpenAPI/Swagger specifications and creates ready-to-use React hooks with built-in loading states, error handling, and authentication.

## ✨ Features

- 🔍 **Automatic API Discovery**: Detects endpoints from OpenAPI/Swagger specs or manual discovery
- ⚛️ **React Hooks Generation**: Creates custom hooks with loading, error, and data states
- 🔐 **Authentication Support**: Token-based, cookie, and session authentication
- 🎭 **Mock Mode**: Generate mock data for offline development
- 📦 **TypeScript Ready**: Generates TypeScript types and interfaces
- 🛠️ **Extensible**: Easy to add support for Vue, Angular, and other frameworks

## 🚀 Quick Start

### Installation

```bash
# Clone the repository
git clone <repository-url>
cd ubf-cli

# Install dependencies
npm install

# Make the CLI globally available
npm link
```

### Basic Usage

```bash
# Connect to a backend API and generate React hooks
ubf connect --backend http://localhost:5000 --frontend react

# With authentication
ubf connect --backend http://localhost:5000 --frontend react --auth token

# With mock mode for offline development
ubf connect --backend http://localhost:5000 --frontend react --mock

# Specify output directory
ubf connect --backend http://localhost:5000 --frontend react --output ./src/api
```

## 📖 Command Reference

### `ubf connect`

Connects to a backend API and generates frontend SDK.

#### Options

| Option | Description | Default | Example |
|--------|-------------|---------|---------|
| `-b, --backend <url>` | Backend API URL | Required | `http://localhost:5000` |
| `-f, --frontend <framework>` | Frontend framework | `react` | `react`, `vue`, `angular` |
| `-o, --output <path>` | Output directory | `./src/api` | `./src/api` |
| `-a, --auth <type>` | Authentication type | `token` | `token`, `cookie`, `session` |
| `-m, --mock` | Enable mock mode | `false` | `--mock` |
| `-s, --spec <path>` | Path to OpenAPI spec file | Auto-detect | `./swagger.json` |

#### Examples

```bash
# Basic connection
ubf connect --backend http://localhost:5000

# With all options
ubf connect \
  --backend http://localhost:5000 \
  --frontend react \
  --output ./src/api \
  --auth token \
  --mock

# Using custom OpenAPI spec
ubf connect \
  --backend http://localhost:5000 \
  --spec ./api-spec.json \
  --frontend react
```

## 🏗️ Generated Code Structure

When you run UBF, it generates the following structure:

```
src/api/
├── index.js          # Main SDK file with all exports
├── apiClient.js      # Axios-based API client with auth
├── types.js          # TypeScript type definitions
└── mockData.js       # Mock data (if mock mode enabled)
```

### Example Generated React Hook

```javascript
import { useState, useEffect, useCallback } from 'react';
import apiClient from './apiClient';

export function useUsers() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await apiClient.get('/users');
      setData(response);
    } catch (err) {
      setError(err.message || 'An error occurred');
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  const refetch = useCallback(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch };
}
```

### Example Generated API Client

```javascript
import axios from 'axios';

class ApiClient {
  constructor(baseURL = 'http://localhost:5000') {
    this.client = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    // Add request interceptor for auth
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('authToken');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );
  }

  // HTTP methods
  async get(url, config = {}) {
    return this.request('GET', url, null, config);
  }

  async post(url, data, config = {}) {
    return this.request('POST', url, data, config);
  }

  // ... other methods
}

export default new ApiClient();
```

## 🔐 Authentication Support

UBF supports multiple authentication methods:

### Token Authentication
```bash
ubf connect --backend http://localhost:5000 --auth token
```
- Automatically adds `Authorization: Bearer <token>` header
- Token stored in `localStorage.getItem('authToken')`

### Cookie Authentication
```bash
ubf connect --backend http://localhost:5000 --auth cookie
```
- Cookies are automatically sent with requests
- No additional configuration needed

### Session Authentication
```bash
ubf connect --backend http://localhost:5000 --auth session
```
- Uses session-based authentication
- Handles CSRF tokens automatically

## 🎭 Mock Mode

Enable mock mode for offline development:

```bash
ubf connect --backend http://localhost:5000 --mock
```

This generates:
- Mock data for all endpoints
- Simulated network delays
- Realistic response structures

## 🧪 Testing with Sample Backend

A sample backend is included for testing:

```bash
# Start the sample backend
cd examples/sample-backend
npm install
npm start

# In another terminal, generate frontend SDK
ubf connect --backend http://localhost:5000 --frontend react --mock
```

The sample backend includes:
- User management endpoints (`/users`)
- Authentication (`/auth/login`)
- Health check (`/health`)
- OpenAPI specification (`/swagger.json`)

## 🛠️ Development

### Project Structure

```
ubf-cli/
├── src/
│   ├── index.js              # CLI entry point
│   ├── commands/
│   │   └── connect.js        # Connect command implementation
│   ├── parsers/
│   │   └── backendParser.js  # OpenAPI/Swagger parser
│   └── generators/
│       ├── reactGenerator.js # React hooks generator
│       └── mockGenerator.js  # Mock data generator
├── examples/
│   └── sample-backend/       # Sample backend for testing
└── README.md
```

### Adding New Frontend Frameworks

To add support for a new frontend framework:

1. Create a new generator in `src/generators/`
2. Implement the `generate()` method
3. Add the framework to the CLI options
4. Update the `getGenerator()` function in `connect.js`

### Adding New Backend Parsers

To add support for a new backend format:

1. Create a new parser in `src/parsers/`
2. Implement the `parseBackend()` method
3. Update the `BackendParser` class to use the new parser

## 📝 API Reference

### BackendParser

```javascript
class BackendParser {
  async parseBackend(config) {
    // Parse backend API and return endpoints
  }
}
```

### ReactGenerator

```javascript
class ReactGenerator {
  async generate(endpoints, config) {
    // Generate React hooks and return SDK code
  }
}
```

### MockGenerator

```javascript
class MockGenerator {
  generateMockResponse(endpoint, method) {
    // Generate mock response for endpoint
  }
}
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details.

## 🆘 Support

- Create an issue for bug reports
- Start a discussion for feature requests
- Check the examples directory for usage examples

---

**Made with ❤️ for developers who want to focus on building great products, not boilerplate API code.**
