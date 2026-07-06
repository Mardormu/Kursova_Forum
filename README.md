# Forum Application

A RESTful forum API built with Spring Boot 4.0.5, featuring user authentication, role-based access control, and topic/reply management.

## Features

- **User Management**: Registration, login, and role-based access (ADMIN, MODERATOR, USER)
- **Topics**: Create, view, and edit topics with unique titles and view counts
- **Replies**: Create and edit replies with pagination (10 per page)
- **Authentication**: JWT-based authentication with configurable expiration
- **Role-Based Security**:
  - **ADMIN**: Can create users, promote to moderator, edit all content
  - **MODERATOR**: Can edit all topics and replies
  - **USER**: Can create topics/replies and edit own content
- **API Documentation**: Scalar UI for interactive API testing
- **Health Checks**: Actuator endpoints for monitoring
- **Maintenance Mode**: Configurable restore mode with allowlist

## Tech Stack

- **Java 21**
- **Spring Boot 4.0.5**
- **Spring Data JPA** with Hibernate
- **PostgreSQL** database
- **Flyway** for database migrations
- **Spring Security** with JWT authentication
- **MapStruct** for DTO mapping
- **OpenAPI 3.0** with Scalar documentation
- **Testcontainers** for integration testing
- **Lombok** for reducing boilerplate

## Prerequisites

- Java 21 or higher
- Maven 3.9+
- Docker (for PostgreSQL)

## Installation

### 1. Clone the repository

```bash
git clone <repository-url>
cd forum-mse-2026-main
```

### 2. Configure environment variables

Create a `.env` file in the project root:

```env
APP_RESTORE_ENABLED=false
JWT_SECRET=your-secret-key-at-least-32-characters-long
JWT_EXPIRATION_MS=86400000
POSTGRES_DB=forum
POSTGRES_USER=admin
POSTGRES_PASSWORD=nemski
```

### 3. Start PostgreSQL database

```bash
docker-compose up -d
```

The database will be available at `localhost:5432`.

### 4. Run the application

```bash
# On Unix/Linux/Mac
./mvnw spring-boot:run

# On Windows
mvnw.cmd spring-boot:run
```

The application will start on `http://localhost:9000`.

## API Documentation

Once the application is running, access the interactive API documentation at:

- **Scalar UI**: `http://localhost:9000/scalar`
- **OpenAPI JSON**: `http://localhost:9000/v3/api-docs`

## API Endpoints

### Authentication

- `POST /api/auth/register` - Register a new user (role: USER)
- `POST /api/auth/login` - Login and receive JWT token

### Users

- `GET /api/users` - List all users (ADMIN/MODERATOR only)
- `GET /api/users/{id}` - Get user by ID (ADMIN/MODERATOR or self)
- `POST /api/users` - Create user (ADMIN only)
- `PUT /api/users/{id}` - Update user (ADMIN or self)
- `DELETE /api/users/{id}` - Delete user (ADMIN only)

### Topics

- `GET /api/posts` - List all topics
- `GET /api/posts/{id}` - Get topic by ID (increments view count)
- `POST /api/posts` - Create topic (authenticated users)
- `PUT /api/posts/{id}` - Update topic (author, MODERATOR, or ADMIN)

### Replies

- `GET /api/posts/{postId}/replies` - List replies for a topic (paginated, 10 per page)
- `GET /api/replies/{id}` - Get reply by ID
- `POST /api/posts/{postId}/replies` - Create reply (authenticated users)
- `PUT /api/replies/{id}` - Update reply (author, MODERATOR, or ADMIN)

### Health & Monitoring

- `GET /actuator/health` - Health check
- `GET /actuator/info` - Application info
- `GET /livez` - Liveness probe
- `GET /readyz` - Readiness probe

## Example Usage

### Register a user

```bash
curl -X POST http://localhost:9000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "password": "password123"
  }'
```

### Login

```bash
curl -X POST http://localhost:9000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "password123"
  }'
```

Save the `token` from the response for authenticated requests.

### Create a topic

```bash
curl -X POST http://localhost:9000/api/posts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "title": "My First Topic",
    "content": "This is the content of my topic"
  }'
```

### Get replies for a topic (paginated)

```bash
curl -X GET "http://localhost:9000/api/posts/1/replies?page=0&size=10"
```

## Testing

### Run unit tests

```bash
./mvnw test
```

### Run integration tests

```bash
./mvnw verify
```

Integration tests use Testcontainers with PostgreSQL.

## Database Migrations

Flyway migrations are located in `src/main/resources/db/migration`. The application automatically applies migrations on startup.

## Security

- Passwords are hashed using BCrypt
- JWT tokens are signed with HS256 algorithm
- Role-based access control is enforced at the controller level
- All endpoints except `/api/auth/register` and `/api/auth/login` require authentication

## Maintenance Mode

To enable maintenance mode, set `APP_RESTORE_ENABLED=true` in your `.env` file. When enabled, only allowlisted endpoints are accessible:

- `/livez`
- `/readyz`
- `/actuator/health/**`
- `/actuator/info`
- `/ops/restore/**`


