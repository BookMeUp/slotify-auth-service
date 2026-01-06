# Authentication Service

The Authentication Service handles user registration, login, and JWT token management for the Slotify application.

## Features

- User registration with role-based access (customer/provider)
- Secure password hashing using bcrypt
- JWT token generation and validation
- User authentication and login
- Prometheus metrics exposure
- Health check endpoint

## Tech Stack

- **Framework**: Flask 2.3.2
- **Authentication**: Flask-JWT-Extended
- **Password Hashing**: bcrypt
- **Monitoring**: Prometheus Flask Exporter
- **HTTP Client**: requests
- **Runtime**: Python 3.13

## Prerequisites

- Python 3.13+
- Database Service running (for user data storage)

## Installation

### Local Development

1. Install dependencies:
```bash
pip install -r requirements.txt
```

2. Set environment variables (optional):
```bash
export JWT_SECRET_KEY="your-secret-key"
export DB_SERVICE_URL="http://localhost:5003"
```

3. Run the service:
```bash
python app.py
```

The service will start on `http://localhost:5001`

### Docker

Build and run using Docker:

```bash
docker build -t auth-service .
docker run -p 5001:5001 \
  -e JWT_SECRET_KEY="your-secret-key" \
  -e DB_SERVICE_URL="http://db-service:5003" \
  auth-service
```

Or pull from Docker Hub:

```bash
docker pull alexnv67/auth-service
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `JWT_SECRET_KEY` | Secret key for JWT token generation | `super-secret-key` |
| `DB_SERVICE_URL` | URL of the Database Service | `http://db-service:5003` |

## API Endpoints

### Health Check
```http
GET /auth/health
```

**Response:**
```
Authentication Service is running
```

### Register
```http
POST /auth/register
```

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "securepassword",
  "role": "customer"  // optional, defaults to "customer"
}
```

**Response:** `201 Created`
```json
{
  "message": "User registered successfully"
}
```

### Login
```http
POST /auth/login
```

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "securepassword"
}
```

**Response:** `200 OK`
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "role": "customer"
  }
}
```

### Logout
```http
DELETE /auth/logout
```

**Response:** `200 OK`
```
JWT token cleared
```

## Metrics

Prometheus metrics are automatically exposed at:
```
GET /metrics
```

## Architecture

The Authentication Service is part of a microservices architecture:

```
Client -> Auth Service (Port 5001) -> Database Service (Port 5003) -> PostgreSQL
```

- **Stateless**: Does not maintain user sessions
- **JWT-based**: Uses JSON Web Tokens for authentication
- **Secure**: Passwords are hashed with bcrypt before storage

## CI/CD

The service uses GitHub Actions for continuous deployment:

- **Trigger**: Push to `main` branch
- **Action**: Builds Docker image and pushes to Docker Hub (`alexnv67/auth-service`)
- **Workflow**: `.github/workflows/docker-publish.yml`

## Development Notes

- The service runs in debug mode is disabled in production (`debug=False`)
- All endpoints except health check require valid JSON payloads
- Password complexity validation should be added before production use
- Consider implementing refresh tokens for better security
- Rate limiting should be added to prevent brute-force attacks

## Security Considerations

⚠️ **Important:**
- Change `JWT_SECRET_KEY` in production
- Use strong, unique secrets for JWT
- Implement rate limiting on login endpoint
- Add HTTPS in production
- Consider implementing refresh token rotation
- Add password complexity requirements

## Related Services

- **Database Service**: Handles user data persistence
- **Business Logic Service**: Consumes authentication tokens

## License

Part of the Slotify platform.
