# HealthOS — AI-Powered Health Information Platform

> A full-stack web application that serves as a centralized platform for managing patient health information, with an integrated AI assistant that analyzes medical notes and
 surfaces clinical insights in real time.
---

## What It Does

HealthOS lets clinicians and patients manage health records through a clean dashboard interface. The platform connects a React frontend to an Express REST API backed by MongoDB Atlas, with AWS services handling file storage and AI-powered text analysis.

**Core features:**
- Add and manage patient records with clinical notes
- Run AI analysis on patient notes using Amazon Comprehend — detects medical entities (conditions, medications, anatomy) and extracts key phrases with confidence scores
- Upload medical documents (PDFs, images, lab results) directly to AWS S3
- Persistent data storage via MongoDB Atlas — data survives restarts and is accessible across sessions
- Tabbed dashboard UI with a dark, professional design

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | React 18 + Vite | Dynamic, component-based UI |
| Backend | Node.js + Express | REST API, request routing, middleware |
| Database | MongoDB Atlas + Mongoose | Persistent patient record storage |
| File Storage | AWS S3 | Medical document uploads |
| AI / NLP | Amazon Comprehend | Entity detection and key phrase extraction from clinical notes |
| File Handling | Multer | Multipart file upload parsing |
| Config | dotenv | Environment variable management |

---

## Architecture

```
client/                        # React frontend (Vite)
│
├── src/
│   └── App.jsx                # Main app — patient dashboard, upload UI, AI analysis
│
server/                        # Node.js + Express backend
│
├── models/
│   └── Patient.js             # Mongoose schema — name, notes, createdAt
│
├── index.js                   # Express server — all API routes
└── .env                       # Environment variables (not committed)
```

**Request flow:**

```
Browser (React)
    │
    ├── GET /api/patients       → MongoDB Atlas → return all patient records
    ├── POST /api/patients      → validate → save to MongoDB → return new record
    ├── POST /api/upload        → multer → AWS S3 PutObjectCommand → return file key
    └── POST /api/analyze       → Amazon Comprehend → DetectEntities + DetectKeyPhrases → return insights
```

---

## API Endpoints

### `GET /api/patients`
Returns all patient records from MongoDB.

**Response:**
```json
[
  {
    "_id": "64f...",
    "name": "Jane Doe",
    "notes": "Patient reports fatigue and low iron. Currently taking lisinopril 10mg.",
    "createdAt": "2026-07-18T..."
  }
]
```

---

### `POST /api/patients`
Creates a new patient record.

**Request body:**
```json
{
  "name": "Jane Doe",
  "notes": "Patient reports fatigue and low iron."
}
```

---

### `POST /api/upload`
Uploads a file to AWS S3.

**Request:** `multipart/form-data` with a `file` field.

**Response:**
```json
{
  "message": "File uploaded successfully",
  "key": "uploads/1721308800000-labresult.pdf",
  "filename": "labresult.pdf"
}
```

---

### `POST /api/analyze`
Sends patient notes to Amazon Comprehend for NLP analysis.

**Request body:**
```json
{
  "text": "Patient reports fatigue and low iron. Currently taking lisinopril 10mg daily."
}
```

**Response:**
```json
{
  "entities": [
    { "text": "fatigue", "type": "OTHER", "score": 94 },
    { "text": "lisinopril 10mg", "type": "QUANTITY", "score": 89 }
  ],
  "keyPhrases": [
    "fatigue",
    "low iron",
    "lisinopril 10mg daily"
  ]
}
```

---

## Local Setup

### Prerequisites
- Node.js v18+
- A [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) account (free tier)
- An [AWS account](https://aws.amazon.com) with S3 and Comprehend access (free tier)

### 1. Clone the repo
```bash
git clone https://github.com/BradleyWPL/health-platform.git
cd health-platform/health-platform
```

### 2. Install server dependencies
```bash
cd server
npm install
```

### 3. Configure environment variables
Create a `.env` file inside the `server/` folder:

```env
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/healthplatform?retryWrites=true&w=majority
PORT=5000
AWS_ACCESS_KEY_ID=your_access_key_id
AWS_SECRET_ACCESS_KEY=your_secret_access_key
AWS_REGION=us-east-1
S3_BUCKET_NAME=your-bucket-name
```

### 4. Start the server
```bash
node index.js
# → Connected to MongoDB
# → Server is running on http://localhost:5000
```

### 5. Install and start the client
```bash
cd ../client
npm install
npm run dev
# → Local: http://localhost:5173
```

Open `http://localhost:5173` in your browser.

---

## AWS Services Used

### S3 (Simple Storage Service)
Medical documents uploaded through the Documents tab are stored in an S3 bucket using the AWS SDK v3 `PutObjectCommand`. 
Files are namespaced with a timestamp prefix (`uploads/timestamp-filename`) to prevent collisions.

### Amazon Comprehend
Patient notes are analyzed using two Comprehend API calls, executed in parallel via `Promise.all()`:
- `DetectEntitiesCommand` — identifies named entities in clinical text (conditions, quantities, organizations, dates)
- `DetectKeyPhrasesCommand` — extracts the most meaningful phrases from the note

Results are returned to the React frontend and rendered as color-coded tags on the patient card, color-coded by entity type.

---

## Key Design Decisions

**Why MongoDB over a relational DB?**
Patient records are naturally document-shaped — a patient has a name, freeform notes, and a timestamp. MongoDB's flexible schema means no migrations 
when adding new fields later (e.g. appointments, vitals, medications).

**Why `Promise.all()` for Comprehend?**
`DetectEntities` and `DetectKeyPhrases` are independent calls. Running them sequentially would double the wait time. `Promise.all()` fires both simultaneously 
and resolves when both complete — cutting latency roughly in half.

**Why Multer with `memoryStorage()`?**
Writing files to disk before uploading to S3 adds unnecessary I/O and leaves temp files to clean up. `memoryStorage()` holds the file as a buffer in RAM and streams 
it directly to S3 — faster and cleaner for small-to-medium medical documents.

---

## Future Enhancements

- [ ] JWT-based authentication — separate patient and clinician roles
- [ ] Amazon SageMaker integration — train a custom model on anonymized vitals data for anomaly detection
- [ ] Appointment scheduling module with calendar UI
- [ ] Vital signs chart (heart rate, blood pressure trends over time) using Recharts
- [ ] Medical History timeline view per patient
- [ ] Export patient records as PDF

---

## Author

**Bradley Pierre-Louis**
Computer Science @ Nova Southeastern University | Graduating June 2028
Minors: Health Informatics & Cybersecurity

[GitHub](https://github.com/BradleyWPL) · [LinkedIn](https://linkedin.com/in/BradleyP-L)
