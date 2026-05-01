<p align="center">
  <img src="gmulogo" alt="George Mason University" width="220">
</p>

# Invisible CAPTCHA — Backend

Java Spring Boot API for the Invisible CAPTCHA Engine. The backend receives **behavioral biometric data** (mouse movement and keystroke timing) from the React frontend, forwards it to the ML service for inference, and returns a **human / bot verdict**. Deployed on Tomcat behind Nginx.

This is one of three repos in the senior design project. See the main project README: [RayQCodes/readme](https://github.com/RayQCodes/readme).

---

## What It Does

- Accepts incoming requests from the frontend with behavioral biometric payloads (mouse + keystroke data).
- Validates and prepares the payload for inference.
- Forwards the data to the ML service running on a separate GPU EC2 instance.
- Receives the classification result (human or bot) and returns it to the frontend.
- Handles session and request routing through Spring Boot.

---

## Tech Stack

- **Java** (Spring Boot)
- **Tomcat** as the servlet container
- **Nginx** as the reverse proxy / front door
- **Maven** for build and dependency management

---

## Project Structure

The Spring Boot project lives in a nested folder:

```
Backend/
└── captcha-decider-backend-master/
    └── captcha-decider-backend-master/   <- run commands from here
        ├── src/
        │   └── main/
        │       ├── java/        <- Spring Boot source
        │       └── resources/   <- application.properties, static files
        ├── pom.xml
        └── ...
```

> All Maven commands below should be run from the inner `captcha-decider-backend-master/` folder.

---

## Getting Started

### Prerequisites
- **Java 17** or higher ([download](https://adoptium.net/))
- **Maven 3.8+** ([download](https://maven.apache.org/download.cgi))
- The **ML service** running and reachable (see [RayQCodes/ML-Model](https://github.com/RayQCodes/ML-Model))

### Build

```bash
cd captcha-decider-backend-master/captcha-decider-backend-master
mvn clean install
```

### Run Locally

```bash
mvn spring-boot:run
```

The server starts on **http://localhost:8080** by default.

### Build a Deployable WAR/JAR

```bash
mvn clean package
```

The build output goes to `target/`. Drop the `.war` into Tomcat's `webapps/` folder for production deployment.

---

## Configuration

Configuration lives in `src/main/resources/application.properties`. Key settings to check before running:

- **Server port** — defaults to `8080`
- **ML service URL** — the endpoint of the GPU EC2 instance running the Gemma model
- **CORS** — the frontend origin (`http://localhost:3000` for local dev) must be allowed

---

## API Overview

The frontend posts behavioral biometric data and receives a verdict. Exact endpoint paths live in the controller classes under `src/main/java/`.

Typical flow:

```
POST /api/verify
  body: { mouseEvents: [...], keystrokeEvents: [...] }
  response: { verdict: "human" | "bot", confidence: 0.97 }
```

---

## Deployment

The backend is deployed on **EC2 #1 (Web Server)** alongside the frontend:

```
+---------------------------------------+
|  EC2 #1 — Web Server                  |
|  Nginx -> Tomcat                      |
|    - React frontend                   |
|    - Spring Boot backend (this repo)  |
+---------------------------------------+
              |
              |  inference request
              v
+---------------------------------------+
|  EC2 #2 — GPU Instance                |
|    - Gemma 2-bit + LoRA               |
+---------------------------------------+
```

Nginx reverse-proxies `/api/*` requests to Tomcat (port `8080`), which serves the Spring Boot backend.

---

## Related Repositories

- **Main project README:** [RayQCodes/readme](https://github.com/RayQCodes/readme)
- **Frontend:** [RayQCodes/frontend](https://github.com/RayQCodes/frontend)
- **ML Model:** [RayQCodes/ML-Model](https://github.com/RayQCodes/ML-Model)

---

## Team

George Mason University — Senior Design Capstone CYSE 2026
Pragalv Bhattarai, Raymond Quan, Faizan Munir, Logan Breckenridge, Jeremiah Mccormick, Saipradhith Gudapati

**Sponsor:** Dr. MingKui Wei
**Mentor:** Dr. Jair Ferrari
