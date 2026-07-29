# AI-Powered Placement Preparation and Career Guidance System

A full-stack web application that helps engineering students prepare for placements: aptitude and coding
practice, resume building, mock interviews, and AI-powered (Google Gemini) career recommendations.

**Tech Stack:** React (Vite) + Tailwind CSS · Node.js + Express.js · MySQL · JWT Auth · Google Gemini API

---

## 1. Project Structure

```
placement-system/
├── frontend/                 # React + Vite + Tailwind app
│   ├── src/
│   │   ├── components/       # Navbar, Footer, Sidebar, ProtectedRoute
│   │   ├── context/          # AuthContext, ThemeContext
│   │   ├── layouts/          # DashboardLayout
│   │   ├── pages/            # Home, Login, Register, Dashboard, Aptitude, ...
│   │   └── services/         # api.js (Axios client)
│   ├── package.json
│   └── .env.example
├── backend/                  # Node.js + Express API
│   ├── config/db.js          # MySQL connection pool
│   ├── controllers/          # Route handlers
│   ├── routes/                # Express routers
│   ├── middleware/           # JWT auth, error handler
│   ├── services/geminiService.js  # Gemini AI integration
│   ├── utils/generateToken.js
│   ├── server.js             # App entry point
│   ├── package.json
│   └── .env.example
└── database/
    └── schema.sql            # MySQL schema + sample data
```

---

## 2. Prerequisites

- Node.js 18+ and npm
- MySQL 8+ (or MariaDB) running locally or remotely
- A Google Gemini API key — get one at https://aistudio.google.com/app/apikey
- VS Code (recommended) with the "ES7+ React/Redux/JS Snippets" and "MySQL" extensions (optional but helpful)

---

## 3. Database Setup

1. Start your MySQL server.
2. Run the schema file to create the database, tables, and sample data:

```bash
mysql -u root -p < database/schema.sql
```

This creates the `placement_system` database with all 11 tables (students, skills, student_skills,
aptitude_questions, aptitude_attempts, coding_problems, coding_submissions, career_recommendations,
learning_resources, resumes, mock_interviews, progress, admin_users) and inserts sample questions,
problems, skills, and resources.

3. **Important:** The sample admin row in `schema.sql` has a placeholder password hash. To create a real
   admin login, generate a bcrypt hash and update the row:

```bash
node -e "console.log(require('bcryptjs').hashSync('YourAdminPassword123', 10))"
```

Then in MySQL:
```sql
UPDATE admin_users SET password = '<generated_hash>' WHERE email = 'admin@placementsystem.com';
```

---

## 4. Backend Setup

```bash
cd backend
npm install
cp .env.example .env
```

Edit `.env` with your actual values:

```
PORT=5000
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=placement_system
JWT_SECRET=replace_this_with_a_long_random_secret_key
JWT_EXPIRES_IN=7d
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-1.5-flash
CLIENT_URL=http://localhost:5173
```

Run the backend:

```bash
npm run dev      # with nodemon (auto-restart)
# or
npm start        # plain node
```

The API will be available at `http://localhost:5000/api`. Visit `http://localhost:5000/api/health` to
confirm it's running.

---

## 5. Frontend Setup

Open a **second terminal**:

```bash
cd frontend
npm install
cp .env.example .env
```

Edit `.env` if your backend runs on a different host/port:

```
VITE_API_BASE_URL=http://localhost:5000/api
```

Run the frontend:

```bash
npm run dev
```

Visit `http://localhost:5173` in your browser.

---

## 6. Running the Full Project in VS Code (Step by Step)

1. Open the `placement-system` folder in VS Code (`File > Open Folder`).
2. Open a terminal (`` Ctrl+` ``) and split it into two panels (`Terminal > Split Terminal`).
3. In the first terminal: `cd backend && npm install && npm run dev`
4. In the second terminal: `cd frontend && npm install && npm run dev`
5. Make sure MySQL is running and you've imported `database/schema.sql`.
6. Make sure both `.env` files are filled in (see sections 4 and 5).
7. Open `http://localhost:5173` in your browser — register a new student account and explore the app.
8. To log in as admin, use the email/password you set in step 3 of Database Setup.

---

## 7. API Endpoint Documentation

Base URL: `http://localhost:5000/api`

### Authentication
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/auth/register` | No | Register a new student |
| POST | `/auth/login` | No | Login (student or admin) |
| GET | `/auth/profile` | Yes | Get logged-in user's profile |

### Students
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/students` | Admin | List/search all students |
| GET | `/students/:id` | Yes | Get a student's full profile |
| PUT | `/students/:id` | Yes | Update a student's profile |
| DELETE | `/students/:id` | Admin | Delete a student |

### Skills
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/skills` | No | List all master skills |
| POST | `/skills` | Yes | Add/update a skill for the logged-in student |

### Aptitude
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/aptitude/questions?category=Quantitative&limit=10` | No | Get random questions |
| POST | `/aptitude/submit` | Yes | Submit answers, get score + analysis |

### Coding
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/coding/problems?difficulty=Easy` | No | List coding problems |
| POST | `/coding/submit` | Yes | Submit code for a problem |

### AI (Google Gemini)
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/ai/recommend-career` | Yes | Get career role, skill gaps, roadmap |
| POST | `/ai/skill-gap` | Yes | Compare current skills vs a target role |
| POST | `/ai/resume-feedback` | Yes | Get AI feedback on resume content |
| POST | `/ai/interview-questions` | Yes | Generate mock interview questions |

### Progress
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/progress/:studentId` | Yes | Get module-wise progress + stats |
| POST | `/progress/update` | Yes | Update completion % for a module |

### Resources
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/resources?category=Java` | No | List learning resources |
| POST | `/resources` | Admin | Add a new resource |

All authenticated endpoints require an `Authorization: Bearer <token>` header, where `<token>` is the
JWT returned by `/auth/login` or `/auth/register`.

---

## 8. Notes on the Gemini AI Integration

`backend/services/geminiService.js` wraps all calls to the Gemini API through the official
`@google/generative-ai` SDK. Every AI-facing controller (`aiController.js`) funnels through this file, so
the model name, prompt format, and JSON parsing logic live in one place. If Gemini returns text that
isn't valid JSON (which can happen occasionally with any LLM), the service falls back to returning
`{ raw: text }` so the frontend can still display something instead of erroring out.

## 9. Notes on the Coding Judge

The `/api/coding/submit` endpoint currently **simulates** a "Passed" result to demonstrate the full flow
(this is intentionally beginner-friendly and safe to run without a sandboxed code execution environment).
For production use, replace the logic in `backend/controllers/codingController.js` with a real code
execution service (e.g., Judge0, a Docker-based sandbox, or a serverless code runner).

## 10. Security Notes

- Passwords are hashed with bcrypt before storage — never stored in plain text.
- JWTs are signed with `JWT_SECRET` — use a long, random value in production and never commit `.env`.
- CORS is restricted to `CLIENT_URL` — update this when deploying to a real domain.
- This project is built for learning/demo purposes; add rate limiting, input sanitization, and HTTPS
  before deploying publicly.
