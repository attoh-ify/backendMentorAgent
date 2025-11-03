# 🧠 Backend Mentor Agent

**Backend Mentor Agent** is an AI-powered backend development tutor built using the [Mastra AI framework](https://docs.mastra.ai/).  
It provides structured, conversational lessons, answers questions about backend concepts, and generates daily learning content using **Gemini 2.5 Pro**.

---

## 🚀 Overview

The agent acts as an intelligent mentor that can:

- 📚 Teach backend development in a clear, structured way  
- 🎯 Pick relevant topics or generate new ones automatically  
- 💬 Answer technical questions and expand on related subtopics  
- 🧩 Persist memory about previously covered topics using a SQLite-based store  
- 🌐 Expose a JSON-RPC API endpoint for **A2A (Agent-to-Agent)** communication  

---

## 📂 Project Structure

src/
- ├── mastra/
- │ ├── index.ts # Main Mastra configuration and agent registration
- │ ├── storage.ts # Local LibSQL (SQLite) storage configuration
- │ ├── a2a/
- │ │ └── router.ts # A2A API endpoint handler (JSON-RPC interface)
- │ └── agents/
- │ └── backendMentorAgent.ts # Core agent definition and behavior
- └── package.json


---

## ⚙️ How It Works

### 1. Agent Definition (`backendMentorAgent.ts`)
The **Backend Mentor Agent** is configured with:
- A detailed **instruction prompt** that defines its teaching style and tone.
- A **Gemini 2.5 Pro** model for reasoning and content generation.
- A **LibSQL-based memory store** (`mastra.db`) for topic persistence.

```ts
export const backendMentorAgent = new Agent({
  name: "Backend Mentor",
  model: "google/gemini-2.5-pro",
  instructions: `You are "Backend Mentor" — an AI teacher for backend developers...`,
  memory: new Memory({
    storage: new LibSQLStore({
      url: "file:../mastra.db",
    }),
  }),
});
```

### 2. A2A Router (router.ts)

Defines an API route that lets other services or workflows communicate with the agent via JSON-RPC.

Endpoint:
POST /a2a/agent/:agentId

Example Payload:

```json
{
  "jsonrpc": "2.0",
  "id": "123",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [{ "kind": "text", "text": "Teach me about caching in Node.js" }]
    }
  }
}
```

Example Response:
```json
{
  "jsonrpc": "2.0",
  "id": "123",
  "result": {
    "status": { "state": "completed" },
    "artifacts": [
      {
        "artifactId": "uuid",
        "name": "backendMentorAgentResponse",
        "parts": [{ "kind": "text", "text": "Caching is a method..." }]
      }
    ]
  }
}
```

### 3. Mastra Setup (index.ts)

The Mastra instance:

- Registers the backendMentorAgent

- Uses a local SQLite (:memory: or mastra.db)

- Sets up logging, observability, and the A2A route

```ts
export const mastra = new Mastra({
  agents: { backendMentorAgent },
  storage: new LibSQLStore({ url: ":memory:" }),
  logger: createLogger({
    name: "BackendMentorAgent",
    level: "info",
  }),
  observability: { default: { enabled: true } },
  server: {
    build: { openAPIDocs: true, swaggerUI: true },
    apiRoutes: [a2aAgentRoute],
  },
});
```

## 🧑‍💻 Running Locally
### 1️⃣ Install Dependencies
```bash
npm install
```

If you encounter a lockfile mismatch error, run:

```bash
rm -rf node_modules package-lock.json
npm install
```

### 2️⃣ Start the Agent in Development
```bash
npm run dev
```
This runs Mastra in watch mode and serves the API locally.

### 3️⃣ Build for Production
```bash
npm run build
```

### 4️⃣ Start in Production
```bash
npm start
```
This will launch the Mastra server and expose your A2A API.

🧭 Available API Routes
Method	Route	Description
POST	/a2a/agent/:agentId	Sends a JSON-RPC message to a specific agent
GET	/docs	Swagger UI (auto-generated)
GET	/openapi.json	OpenAPI spec file

## 🧩 Environment Details
- Runtime: Node.js 20+
- Framework: Mastra
- Database: SQLite (via LibSQL)
- Model: Google Gemini 2.5 Pro
- Memory: Persistent topic tracking

## 💡 Example Usage
To get a lesson:
```bash
curl -X POST http://localhost:3000/a2a/agent/backendMentorAgent \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc": "2.0",
  "id": "001",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [{"kind": "text", "text": "Teach me about REST APIs"}]
    }
  }
}'
```

## 🧾 License

MIT License © 2025 — Backend Mentor Agent