# IoT-Telemetry-Ingestor_vihanga
# IoT Telemetry Ingestor

A NestJS-based IoT telemetry ingestion system with MongoDB storage, Redis caching, and real-time alerting.

## Features

- ✅ REST API for telemetry ingestion
- ✅ MongoDB Atlas persistence
- ✅ Redis caching for latest device readings
- ✅ Threshold-based alerting via webhook
- ✅ Site-level analytics with aggregation
- ✅ Bearer token authentication
- ✅ 60-second alert deduplication
- ✅ Health checks for MongoDB and Redis
- ✅ Comprehensive validation and error handling

## Setup

### Prerequisites
- Node.js 18+ and npm
- MongoDB Atlas account (free tier)
- Redis (Docker or local installation)
- webhook.site URL for alerts

### Installation

1. Clone the repository
```bash
git clone <repo-url>
cd telemetry-ingestor
```

2. Install dependencies
```bash
npm install
```

3. Configure environment variables
Create a `.env` file based on `.env.example`:
```env
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/telemetry?retryWrites=true&w=majority
REDIS_URL=redis://localhost:6379
ALERT_WEBHOOK_URL=https://webhook.site/YOUR-UNIQUE-ID
INGEST_TOKEN=secret123
PORT=3000
```

4. Start Redis
```bash
docker run -d -p 6379:6379 redis:alpine
```

5. Run the application
```bash
# Development
npm run start:dev

# Production
npm run build
npm run start:prod
```

## API Endpoints

### POST /api/v1/telemetry
Ingest telemetry data (single or array).

**Headers:**
```
Authorization: Bearer secret123
Content-Type: application/json
```

**Body:**
```json
{
  "deviceId": "dev-001",
  "siteId": "site-A",
  "ts": "2025-10-31T10:00:00.000Z",
  "metrics": {
    "temperature": 45.2,
    "humidity": 65
  }
}
```

**Response:** `201 Created`

### GET /api/v1/devices/:deviceId/latest
Get latest reading for a device (Redis → MongoDB fallback).

**Response:**
```json
{
  "deviceId": "dev-001",
  "siteId": "site-A",
  "ts": "2025-10-31T10:00:00.000Z",
  "metrics": {
    "temperature": 45.2,
    "humidity": 65
  }
}
```

### GET /api/v1/sites/:siteId/summary
Get aggregated statistics for a site.

**Query Parameters:**
- `from`: ISO timestamp (required)
- `to`: ISO timestamp (required)

**Response:**
```json
{
  "count": 150,
  "avgTemperature": 42.5,
  "maxTemperature": 55.0,
  "avgHumidity": 68.3,
  "maxHumidity": 92.0,
  "uniqueDevices": 5
}
```

### GET /health
Check MongoDB and Redis connectivity.

**Response:**
```json
{
  "status": "healthy",
  "mongo": true,
  "redis": true
}
```

## Alerting

Alerts are triggered when:
- Temperature > 50°C → `HIGH_TEMPERATURE`
- Humidity > 90% → `HIGH_HUMIDITY`

Alerts are sent to the configured webhook URL with 60-second deduplication per device+reason.

**Webhook.site URL:** `https://webhook.site/YOUR-UNIQUE-ID`
_(Check this URL to see incoming alerts)_

## Testing
```bash
# Unit tests
npm run test

# E2E tests
npm run test:e2e

# Test coverage
npm run test:cov
```

## Quick Verification
```bash
# Ingest data with alert
curl -X POST http://localhost:3000/api/v1/telemetry \
  -H "Authorization: Bearer secret123" \
  -H "Content-Type: application/json" \
  -d '{"deviceId":"dev-002","siteId":"site-A","ts":"2025-10-31T10:00:30.000Z","metrics":{"temperature":51.2,"humidity":55}}'

# Get latest reading
curl http://localhost:3000/api/v1/devices/dev-002/latest

# Get site summary
curl "http://localhost:3000/api/v1/sites/site-A/summary?from=2025-10-31T00:00:00.000Z&to=2025-11-01T00:00:00.000Z"

# Health check
curl http://localhost:3000/health
```

## AI Assistance Usage

This project was built with AI assistance. Here's how AI was used:

1. **Architecture Planning**: Used AI to generate the initial NestJS project structure and recommend best practices for IoT data ingestion patterns.

2. **Code Generation**: AI generated boilerplate code for DTOs, schemas, services, and controllers. I modified validation rules and added custom business logic for alert deduplication.

3. **MongoDB Aggregation**: AI helped craft the complex aggregation pipeline for site summary statistics. I adjusted the pipeline to add compound indexes for better performance.

4. **Error Handling**: AI suggested comprehensive try-catch blocks and logging patterns. I customized error messages and added structured logging.

5. **Testing Strategy**: AI provided test templates for both unit and E2E tests. I extended coverage to include edge cases like invalid payloads and auth failures.

## MongoDB Atlas Configuration

- **Cluster:** Free M0 tier
- **Database:** telemetry
- **Collection:** telemetries (auto-created)
- **Indexes:** Compound indexes on `siteId + ts` and single indexes on `deviceId` for optimal query performance

## Security Features

- Bearer token authentication on ingest endpoint
- Payload size limits (1MB)
- DTO validation with whitelist
- No secrets logged
- Request timeouts (5s for webhooks)
- Input sanitization via class-validator

## Project Structure
```
telemetry-ingestor/
├── src/
│   ├── guards/
│   │   └── auth.guard.ts
│   ├── health/
│   │   ├── health.controller.ts
│   │   └── health.module.ts
│   ├── telemetry/
│   │   ├── dto/
│   │   │   └── telemetry.dto.ts
│   │   ├── schemas/
│   │   │   └── telemetry.schema.ts
│   │   ├── telemetry.controller.ts
│   │   ├── telemetry.module.ts
│   │   └── telemetry.service.ts
│   ├── app.module.ts
│   └── main.ts
├── test/
│   └── app.e2e-spec.ts
├── .env
├── .env.example
└── README.md
```

## License

MIT
