# Notes REST API

A microservices-based REST API for managing notes and notebooks, built with Node.js, Express, MongoDB, and Docker.

## Architecture

This application follows a microservices architecture with the following components:

- **Notes Backend**: Manages individual notes with CRUD operations
- **Notebooks Backend**: Manages notebook collections that can contain notes
- **Reverse Proxy**: Nginx-based proxy for routing requests to appropriate services
- **MongoDB**: Database for data persistence (separate instances for each service)

```
┌─────────────────┐    ┌─────────────────┐
│  Client/Browser │    │   Port :8080    │
└─────────┬───────┘    └─────────┬───────┘
          │                      │
          └──────────────────────┘
                     │
              ┌──────▼──────┐
              │    Nginx    │
              │   Proxy     │
              └──────┬──────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
   ┌────▼───┐   ┌────▼───┐   ┌────▼────┐
   │ Notes  │   │Notebooks│   │ MongoDB │
   │Service │   │Service  │   │ Cluster │
   │ :3000  │   │ :3001   │   └─────────┘
   └────────┘   └─────────┘
```

## Tech Stack

- **Backend**: Node.js, Express.js
- **Database**: MongoDB with Mongoose ODM
- **Proxy**: Nginx
- **Containerization**: Docker & Docker Compose
- **Development**: Nodemon for hot reloading

## API Endpoints

### Notes Service (`/api/notes`)

- `GET /api/notes` - Get all notes
- `GET /api/notes/:id` - Get a specific note
- `POST /api/notes` - Create a new note
- `PUT /api/notes/:id` - Update a note
- `DELETE /api/notes/:id` - Delete a note

**Note Model:**
```json
{
  "title": "string (required)",
  "content": "string (required)",
  "notebookId": "ObjectId (optional)"
}
```

### Notebooks Service (`/api/notebooks`)

- `GET /api/notebooks` - Get all notebooks
- `GET /api/notebooks/:id` - Get a specific notebook
- `POST /api/notebooks` - Create a new notebook
- `PUT /api/notebooks/:id` - Update a notebook
- `DELETE /api/notebooks/:id` - Delete a notebook

**Notebook Model:**
```json
{
  "name": "string (required)",
  "description": "string (optional)"
}
```

## Prerequisites

- Docker and Docker Compose
- Node.js 18+ (for local development)
- MongoDB (handled by Docker Compose)

## Getting Started

### 1. Clone the Repository

```bash
git clone <repository-url>
cd notes-rest-api
```

### 2. Start the Application

```bash
# Start all services (development stage only for faster builds)
docker compose up

# Start in detached mode
docker compose up -d

# Start with rebuild (rebuilds development stage only)
docker compose up --build

# Start with file watching (auto-restart on changes)
docker compose up --watch
```

The application will be available at:
- **API Gateway**: http://localhost:8080
- **Notes API**: http://localhost:8080/api/notes
- **Notebooks API**: http://localhost:8080/api/notebooks

### 3. Stop the Application

```bash
# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v
```

## Development

### Local Development Setup

1. **Install dependencies for each service:**

```bash
# Notes backend
cd notes-backend
npm install

# Notebooks backend
cd ../notebooks-backend
npm install
```

2. **Set up environment variables:**

Create `.env` files in each backend directory with:
```env
PORT=3000  # 3001 for notebooks
DB_URL=mongodb://localhost:27017/notes  # or notebooks
NOTEBOOKS_API_URL=http://localhost:3001  # for notes service
```

3. **Run services individually:**

```bash
# Terminal 1 - Notes service
cd notes-backend
npm run dev

# Terminal 2 - Notebooks service
cd notebooks-backend
npm run dev
```

### Docker Development

The `compose.override.yaml` file is used to modify the base Docker Compose configuration for development purposes without changing the production settings. This follows Docker Compose's standard convention where override files are automatically merged with the base configuration.

**Why use compose.override.yaml?**
- **Network Integration**: Connects microservices to the shared `notes-app-net` network for inter-service communication
- **Environment Separation**: Keeps development-specific configurations separate from production settings  
- **Service Discovery**: Enables services to communicate using service names (e.g., `http://notebooks/api/notebooks`)
- **Hot Reloading**: Can include development-specific volume mounts and watch configurations
- **Team Collaboration**: Developers get consistent development environment without modifying base files
- **Multi-stage Builds**: Targets only the `development` stage of Dockerfiles, skipping production optimization steps for faster builds

**Development vs Production Dockerfile Stages:**
- **Stage 1 (development)**: Includes dev dependencies, nodemon for hot reloading, and full source code
- **Stage 2 (prod-dependencies)**: Installs only production dependencies  
- **Stage 3 (production)**: Uses distroless image for security and minimal footprint

For development with hot reloading:

```bash
# Docker Compose automatically includes compose.override.yaml (development stage only)
docker compose up

# Or explicitly specify both files
docker compose -f compose.yaml -f compose.override.yaml up

# Force rebuild development stage only
docker compose up --build

# With file watching for live reload
docker compose up --watch
```

## Testing the API

### Create a Notebook

```bash
curl -X POST http://localhost:8080/api/notebooks \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My First Notebook",
    "description": "A notebook for important notes"
  }'
```

### Create a Note

```bash
curl -X POST http://localhost:8080/api/notes \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Important Note",
    "content": "This is the content of my note",
    "notebookId": "NOTEBOOK_ID_FROM_PREVIOUS_STEP"
  }'
```

### Get All Notes

```bash
curl http://localhost:8080/api/notes
```

### Get All Notebooks

```bash
curl http://localhost:8080/api/notebooks
```

## Project Structure

```
notes-rest-api/
├── compose.yaml              # Main Docker Compose file
├── compose.override.yaml     # Development overrides
├── notes-backend/
│   ├── compose.yaml         # Notes service configuration
│   ├── Dockerfile
│   ├── package.json
│   ├── db-config/
│   │   └── mongo-init.js    # Database initialization
│   └── src/
│       ├── models.js        # Note data model
│       ├── routes.js        # API routes
│       └── server.js        # Express server
├── notebooks-backend/
│   ├── compose.yaml         # Notebooks service configuration
│   ├── Dockerfile
│   ├── package.json
│   ├── db-config/
│   │   └── mongo-init.js    # Database initialization
│   └── src/
│       ├── models.js        # Notebook data model
│       ├── routes.js        # API routes
│       └── server.js        # Express server
└── reverse-proxy/
    └── nginx.conf           # Nginx configuration
```

## Features

- ✅ RESTful API design
- ✅ Microservices architecture
- ✅ MongoDB integration with Mongoose
- ✅ Docker containerization
- ✅ Nginx reverse proxy
- ✅ Development hot reloading
- ✅ Cross-service communication
- ✅ Input validation and error handling

## Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 8080, 3000, and 3001 are available
2. **Docker issues**: Try `docker compose down -v` and `docker compose up --build`
3. **Database connection**: Check MongoDB container logs with `docker compose logs mongodb`

### Useful Commands

```bash
# View logs
docker compose logs -f

# View specific service logs
docker compose logs -f notes
docker compose logs -f notebooks

# Rebuild specific service
docker compose up --build notes

# Access MongoDB
docker compose exec notes-db mongosh
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the ISC License.