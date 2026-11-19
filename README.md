# 🎯 InterviewIQ - AI-Powered Interview Preparation Platform

[![Live Demo](https://img.shields.io/badge/Demo-Live%20Website-success?style=for-the-badge&logo=vercel)](https://interview-iq-three-xi.vercel.app/)
[![React](https://img.shields.io/badge/React-19.2-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Google Gemini](https://img.shields.io/badge/Google%20Gemini-2.5%20Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)](https://ai.google.dev/)
[![BullMQ](https://img.shields.io/badge/BullMQ-Redis%20Queue-red?style=for-the-badge&logo=redis&logoColor=white)](https://bullmq.io/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)

> **InterviewIQ** is a full-stack, enterprise-grade AI career copilot that parses candidate resumes, compares them against target job descriptions, and synthesizes tailored interview preparation strategies, technical and behavioral assessments, and study plans powered by **Google Gemini 2.5 Flash**.

---

## 🌟 Live Demo

Experience the live application here:  

---

## ⚡ Key Features

- 📄 **Smart Resume Parsing**: Direct PDF parsing extracts candidate work history, education, and technical stack using `pdf-parse`.
- 🎯 **Target Job Matching & Gap Analysis**: Benchmarks candidate qualifications against target job descriptions, computing an objective match score with granular skill gaps and actionable recommendations.
- ⚡ **Asynchronous Job Queue Processing**: Leverages **BullMQ** and **Redis** to offload computationally heavy AI inference and document parsing to background workers. The API immediately responds with `201 Accepted`, preventing HTTP request timeouts and handling high-concurrency workloads seamlessly.
- 🤖 **Resilient Multi-Model AI Fallback**: Integrates Google Generative AI with automated cascading failover (`gemini-2.5-flash` ➔ `gemini-2.0-flash` ➔ `gemini-1.5-flash` ➔ `gemini-flash-latest`) alongside a structured safety fallback, guaranteeing high availability even under upstream rate limits.
- 🧠 **Tailored Technical & Behavioral Roadmaps**: Synthesizes real-world technical interview problems with intention & sample answers, behavioral scenarios mapped to the STAR method, and structured day-by-day study calendars.
- 📑 **Resume PDF Generation**: Exports customized PDF summaries based on candidate self-description and job alignment.
- 🛡️ **Enterprise-Grade Security**:
  - JWT authentication with secure HTTP-only cookies and token blacklisting backed by TTL indexes in MongoDB.
  - Rate limiting via `express-rate-limit` to defend against brute-force and DDoS attacks.
  - HTTP header hardening with `Helmet` and strict schema validation via `Zod`.
  - Structured, production-ready logging with `Winston`.
- 💫 **Modern, High-Performance Frontend**:
  - Built with **React 19**, **Vite**, and **React Router v7**.
  - **TanStack Query (React Query v5)** for optimistic UI updates, automated background polling of queued jobs, and smart cache invalidation.
  - Polished responsive styling with **SCSS** and shimmer loading skeletons.

---

## 🏗️ System Architecture

```text
  ┌─────────────────────────┐
  │  Client (React 19 /     │
  │  TanStack Query v5)     │
  └───────────┬─────────────┘
              │ 1. Upload Resume PDF + Job Description
              ▼
  ┌─────────────────────────┐
  │ Express API Gateway     │
  │ (Helmet, Rate Limiter,  │
  │  Zod Validation, JWT)   │
  └───────────┬─────────────┘
              │ 2. Enqueue Job & return 201 Accepted
              ▼
  ┌─────────────────────────┐         ┌─────────────────────────┐
  │  BullMQ Task Queue      ├────────►│  Redis Server           │
  └───────────┬─────────────┘         └─────────────────────────┘
              │ 3. Background Job Pickup
              ▼
  ┌─────────────────────────┐
  │  Worker Service         │
  │  - Extract PDF text     │
  │  - AI Service Engine    │
  └───────────┬─────────────┘
              │ 4. Prompt with JSON Schema Enforcement
              ▼
  ┌──────────────────────────────────────────────────────────┐
  │  Google Generative AI (Gemini Flash Family)              │
  │  [2.5-flash ──(failover)──► 2.0-flash ──► 1.5-flash]     │
  └───────────────────────────┬──────────────────────────────┘
                              │ 5. Save Completed Report
                              ▼
                  ┌────────────────────────┐
                  │ MongoDB Database       │
                  └───────────┬────────────┘
                              │ 6. Polled by Client
                              ▼
                  ┌────────────────────────┐
                  │ Render Interactive UI  │
                  └────────────────────────┘
```

---

## 🛠️ Tech Stack

| Domain | Technologies |
| :--- | :--- |
| **Frontend** | React 19, Vite, TanStack Query v5, React Router v7, Axios, jsPDF, SCSS |
| **Backend** | Node.js (18+), Express 5, BullMQ, ioredis, Mongoose, Zod, Multer, pdf-parse |
| **Database & Cache** | MongoDB (Data persistence & TTL token blacklisting), Redis (Task Queue) |
| **AI Engine** | Google Generative AI SDK (`@google/generative-ai`), Gemini 2.5 Flash / 2.0 / 1.5 |
| **Security & Logging** | Helmet, express-rate-limit, jsonwebtoken, bcryptjs, Winston Logger |
| **DevOps & Testing** | Docker, Docker Compose, Jest, Supertest, MongoDB Memory Server |

---

## 📁 Repository Structure

```text
Interview-IQ/
├── Backend/
│   ├── src/
│   │   ├── config/          # Database (MongoDB) and Redis connections
│   │   ├── controllers/     # Authentication & Interview business logic
│   │   ├── middlewares/     # Auth, Multer (file upload), Rate Limiting, Error Handlers
│   │   ├── models/          # Mongoose models (User, InterviewReport, BlacklistToken)
│   │   ├── queues/          # BullMQ queue instances
│   │   ├── routes/          # Express route definitions
│   │   ├── services/        # Gemini AI prompt engine & schema enforcement
│   │   ├── utils/           # Winston logger & helper functions
│   │   └── workers/         # BullMQ async background worker
│   ├── tests/               # Automated unit & integration tests (Jest, Supertest)
│   ├── Dockerfile           # Backend containerization spec
│   ├── server.js            # Server entrypoint
│   └── package.json
├── Frontend/
│   ├── src/
│   │   ├── components/      # Common UI components (Navbar, Layout)
│   │   ├── features/        # Feature slices (Auth & Interview flows)
│   │   ├── style/           # SCSS styles and global mixins
│   │   ├── App.jsx          # App root component with React Query Provider
│   │   └── main.jsx         # Client bootstrap
│   ├── vite.config.js       # Vite bundler configuration
│   └── package.json
├── docker-compose.yml       # Multi-container orchestration (Backend + Redis)
├── render.yaml              # Render deployment configuration
└── README.md
```

---

## 🚀 Getting Started

### 1. Prerequisites
Ensure you have the following installed:
- [Node.js](https://nodejs.org/) (v18 or higher)
- [MongoDB](https://www.mongodb.com/) (local instance or MongoDB Atlas)
- [Redis](https://redis.io/) (local instance or via Docker)
- [Google Gemini API Key](https://aistudio.google.com/)

---

### 2. Backend Setup

1. **Navigate to the Backend directory and install dependencies**:
   ```bash
   cd Backend
   npm install
   ```

2. **Configure Environment Variables**:
   Create a `.env` file inside `Backend/` (or copy `.env.example`):
   ```env
   PORT=3000
   MONGO_URI=mongodb://localhost:27017/interview-iq
   JWT_SECRET=your_super_secret_jwt_key
   GEMINI_API_KEY=your_gemini_api_key
   REDIS_URL=redis://localhost:6379
   FRONTEND_URL=http://localhost:5173
   NODE_ENV=development
   ```

3. **Start the Development Server & Worker**:
   ```bash
   npm run dev
   ```
   *The server runs by default on `http://localhost:3000` with the BullMQ worker active.*

4. **Run Automated Tests**:
   ```bash
   npm test
   ```

---

### 3. Frontend Setup

1. **Navigate to the Frontend directory and install dependencies**:
   ```bash
   cd Frontend
   npm install
   ```

2. **Launch the Vite Development Server**:
   ```bash
   npm run dev
   ```
   *The client will be accessible at `http://localhost:5173`.*

---

### 4. Docker Quickstart (Full Stack Backend + Redis)

Spin up the entire backend ecosystem with a single command:
```bash
docker-compose up -d
```
This deploys:
- **Redis Service**: Running on port `6379`
- **Backend API & Queue Worker**: Connected to Redis with auto-restart

---

## 📡 API Reference

### 🔐 Authentication (`/api/auth`)
| Method | Endpoint | Description | Auth Required |
| :--- | :--- | :--- | :---: |
| `POST` | `/api/auth/register` | Register new user account | ❌ |
| `POST` | `/api/auth/login` | Authenticate user & issue JWT | ❌ |
| `GET` | `/api/auth/logout` | Invalidate token and clear session | ✅ |
| `GET` | `/api/auth/profile` | Retrieve authenticated user profile | ✅ |

### 🎯 Interview Preparation (`/api/interview`)
| Method | Endpoint | Description | Auth Required |
| :--- | :--- | :--- | :---: |
| `POST` | `/api/interview/` | Upload resume & job description to enqueue report generation | ✅ |
| `GET` | `/api/interview/` | List all interview reports for authenticated user | ✅ |
| `GET` | `/api/interview/report/:interviewId` | Retrieve interview report details (polls during processing) | ✅ |
| `POST` | `/api/interview/resume/pdf/:interviewReportId` | Generate downloadable tailored PDF | ✅ |

---

## 📈 Scalability & Reliability Design

1. **Zero HTTP Timeouts**: AI prompt generation and document ingestion can take 10-25 seconds. InterviewIQ immediately acknowledges incoming jobs with `201 Accepted` and returns a job ID. The frontend queries the backend via TanStack Query polling until state flips to `completed`.
2. **Cascading Model Redundancy**: If Gemini 2.5 Flash encounters rate limiting (HTTP 429) or transient API failures (HTTP 503), the engine automatically tries Gemini 2.0 Flash and Gemini 1.5 Flash before executing a graceful degradation schema.
3. **Optimized Token Invalidation**: Blacklisted tokens are saved in MongoDB with native MongoDB TTL expiration, self-purging expired session keys automatically without manual database maintenance.

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:
1. Fork the Project.
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`).
3. Commit your Changes (`git commit -m 'feat: Add AmazingFeature'`).
4. Push to the Branch (`git push origin feature/AmazingFeature`).
5. Open a Pull Request.

---

## 👨‍💻 Author

**Yash Chopade**  

