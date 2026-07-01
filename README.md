# Dream Vacation App (Stage 4 Task)
Dream Vacation Destinations

This application allows users to create a list of countries they'd like to visit, providing basic information about each country. The project is structured to mimic a real-life production environment, employing best practices in software development, deployment, and continuous integration/continuous delivery (CI/CD).

## Project Overview

The Dream Vacation App is a full-stack containerized application that allows users to:

Add countries to a dream vacation list
Retrieve country details (capital, population, region)
Store and manage destinations in a PostgreSQL database
Remove unwanted destinations

The project is fully containerized using Docker and Docker Compose, simulating a production-like microservices architecture.

## Tech Stack
Frontend: React + Nginx
Backend: Node.js + Express
Database: PostgreSQL
External API: REST Countries API
Containerization: Docker + Docker Compose
CI/CD: To be implemented (GitHub Actions / Jenkins / Azure DevOps)
Infrastructure as Code: Planned (Terraform / Helm)
⚙️ Setup
🐘 Backend
Navigate to the backend directory.

Run:

npm install
Set up your PostgreSQL database and update the .env file with your database URL.

Start the server:

npm start
🌐 Frontend
Navigate to the frontend directory.

Run:

npm install

Update .env with API URL:

REACT_APP_API_URL=http://localhost:3001

Start the frontend:

npm start
🐳 Docker Setup (Recommended)

Run the full application using Docker Compose:

docker-compose up --build
Access Services:
Frontend → http://localhost:3000
Backend → http://localhost:3001/api/destinations
Database → PostgreSQL on port 5432
✨ Features
Add Countries: Users can add countries to their dream vacation list.
View Country Details: Displays capital, population, and region information.
Remove Countries: Delete saved destinations.
Production-Ready Architecture: Designed with scalability and maintainability in mind.
🧪 Roadmap
CI/CD pipeline automation using GitHub Actions or Jenkins
Infrastructure as Code (Terraform / Helm)
Multi-environment support (dev, staging, production)
Security improvements using secrets management
Microservices decomposition for scalability
🌍 External API Integration

This project uses the REST Countries API to fetch country information.

Example:

GET https://restcountries.com/v3.1/name/Ghana
⚠️ Important Note: External API Deprecation

During runtime testing inside the Docker container, the REST Countries API (v3) returned a deprecation response:

```{
  "success": false,
  "data": null,
  "errors": [
    {
      "message": "This API version has been deprecated. Please visit https://restcountries.com/docs/legacy-api-deprecation to migrate to our new version (v5)."
    }
  ]
}```
Impact:
Backend expects response.data[0]
API now returns a different structure due to deprecation
This leads to runtime parsing errors in capital, region, and population
Conclusion:
Docker networking, backend, frontend, and database integration are fully functional
Issue is strictly due to external API version deprecation (outside project scope)
📦 Docker Architecture

The system consists of:

frontend → React app served via Nginx
backend → Express API service
db → PostgreSQL database

All services communicate through a custom Docker bridge network, with persistent database storage using Docker volumes.

💾 Persistence

PostgreSQL data is persisted using Docker volumes:

volumes:
  postgres-data:
📌 Best Practices
Version control using Git
Environment-based configuration management
Separation of frontend and backend services
Containerized deployment for portability
Documentation for maintainability and onboarding
📌 Summary

This project demonstrates:

Multi-container Docker architecture
Backend–database service communication via Docker networking
Persistent PostgreSQL storage
External API integration
Production-style full-stack application design