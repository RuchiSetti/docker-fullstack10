# RentWise – Apartment Management System

A full-stack web application for managing rental apartments — built with **React + Vite** on the frontend and **Spring Boot** on the backend, backed by **MySQL**. The whole stack runs in Docker containers and is tied together with a **Docker Compose** setup, with a GitHub Actions pipeline handling automatic image builds and pushes to DockerHub.

---

## What This App Does

RentWise is a simple rental property tracker where you can:

- Add, edit, and delete apartment listings
- Track each apartment's name, location, and monthly rent
- Toggle availability status between **Available** and **Occupied** with a single click
- View all listings in a clean, responsive table
- Supports both light and dark mode out of the box

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite 7, Axios |
| Backend | Spring Boot 3.5, Spring Data JPA |
| Database | MySQL 8 |
| Containerization | Docker (multi-stage builds), Docker Compose |
| CI/CD | GitHub Actions |

---

## Project Structure

```
├── react-frontend/             # React + Vite frontend
│   ├── src/
│   │   ├── components/         # ApartmentManager UI component
│   │   └── App.jsx             # Root component
│   └── frontend.Dockerfile     # Multi-stage Docker build for frontend
│
├── SPRINGBOOT/                 # Spring Boot backend
│   ├── src/main/java/com/apartment/
│   │   ├── controller/         # REST API endpoints
│   │   ├── entity/             # Apartment JPA entity
│   │   ├── repository/         # Spring Data JPA repository
│   │   └── service/            # Business logic
│   └── backend.Dockerfile      # Multi-stage Docker build for backend
│
├── docker-compose.yml          # Runs MySQL + backend + frontend together
│
└── .github/workflows/
    └── docker-image.yml        # CI pipeline to build and push Docker images
```

---

## Running With Docker Compose (Recommended)

This is the easiest way to get everything up and running. You just need Docker installed.

```bash
docker-compose up
```

That single command spins up three containers: MySQL, the Spring Boot backend, and the React frontend. Once they're all up, open your browser at **http://localhost:3000**.

To stop everything:

```bash
docker-compose down
```

### Port mapping at a glance

| Service | Container Port | Host Port |
|---|---|---|
| Frontend (Nginx) | 80 | 3000 |
| Backend (Spring Boot) | 2003 | 2025 |
| MySQL | 3306 | 3307 |

---

## Running Locally (Without Docker)

### Prerequisites

- Node.js 20+
- Java 21
- Maven
- MySQL running locally

### Backend

1. Create a MySQL database named `apartment_DB`
2. Update the datasource in `SPRINGBOOT/src/main/resources/application.properties`:
   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/apartment_DB
   spring.datasource.username=root
   spring.datasource.password=docker
   ```
3. Start the backend:
   ```bash
   cd SPRINGBOOT
   ./mvnw spring-boot:run
   ```
   The server starts on **port 2003**.

### Frontend

1. Update the API URL in `react-frontend/.env`:
   ```env
   VITE_API_URL=http://localhost:2003
   ```
2. Install dependencies and start:
   ```bash
   cd react-frontend
   npm install
   npm run dev
   ```
   The app opens at **http://localhost:5173**.

---

## Building Docker Images Manually

### Backend

```bash
cd SPRINGBOOT
docker build -f backend.Dockerfile -t apartment-backend .
```

### Frontend

```bash
cd react-frontend
docker build -f frontend.Dockerfile -t apartment-frontend .
```

The frontend Dockerfile builds the Vite app in a Node container, then copies the static output into an Nginx image — so the final image is lean and serves the built files directly.

---

## CI/CD Pipeline

Every push to `main` (and any pull request) triggers the GitHub Actions workflow, which:

1. Checks out the code
2. Sets up Docker Buildx
3. Logs into DockerHub using repository secrets
4. Builds and pushes the **backend** image as `<your-username>/apartment-backend:latest`
5. Builds and pushes the **frontend** image as `<your-username>/apartment-frontend:latest`

### Setting up secrets

Go to your GitHub repo → **Settings → Secrets → Actions** and add:

| Secret | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Your DockerHub username |
| `DOCKERHUB_TOKEN` | Your DockerHub access token |

---

## API Endpoints

All endpoints are under `/api/apartments`:

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/apartments` | Get all apartments |
| GET | `/api/apartments/{id}` | Get a single apartment by ID |
| POST | `/api/apartments` | Add a new apartment |
| PUT | `/api/apartments/{id}` | Update an existing apartment |
| DELETE | `/api/apartments/{id}` | Delete an apartment |
| PATCH | `/api/apartments/{id}/toggle-status` | Toggle status between Available / Occupied |
| GET | `/api/apartments/health` | Health check endpoint |

---

## Apartment Data Model

Each apartment record stores:

| Field | Type | Description |
|---|---|---|
| `id` | int | Auto-generated primary key |
| `name` | String | Apartment name |
| `location` | String | Address or area |
| `rent` | double | Monthly rent amount |
| `status` | String | `Available` or `Occupied` |

---

## Environment Notes

- The backend connects to MySQL using the hostname `mysqldb` — this works when running inside Docker Compose. Change it to `localhost` for local development.
- The frontend reads `VITE_API_URL` from the `.env` file at build time. When running via Docker Compose, the frontend calls the backend through the host machine on port `2025`.
- MySQL data is persisted using a named Docker volume (`mysql_data`), so your data survives container restarts.
