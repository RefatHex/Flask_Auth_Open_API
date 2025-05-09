# API Manual Testing Guide

This document provides step-by-step instructions to manually test the API, starting from creating an account to performing chat-related requests. Each API is described with its purpose, workflow, and implementation details.

---

## Prerequisites

1. Install a tool like [Postman](https://www.postman.com/) or [cURL](https://curl.se/).
2. Obtain the base URL for the API (e.g., `http://localhost:5000`).

---

## API Endpoints

### 1. **Create an Account**

**Endpoint:** `POST /signup`

**Description:**  
Registers a new user account. Only the admin email (defined in `.env` as `ADMIN_EMAIL`) is allowed to register.

**Request JSON:**

```json
{
  "name": "Test User",
  "email": "admin@example.com",
  "password": "Test@1234",
  "confirm": "Test@1234"
}
```

**Example cURL Command:**

```bash
curl -X POST http://localhost:5000/signup \
-H "Content-Type: application/json" \
-d '{
  "name": "Test User",
  "email": "admin@example.com",
  "password": "Test@1234",
  "confirm": "Test@1234"
}'
```

**Response:**

```json
{
  "msg": "User created successfully"
}
```

---

### 2. **Login**

**Endpoint:** `POST /login`

**Description:**  
Authenticates the user and returns a JWT token for accessing protected resources.

**Request JSON:**

```json
{
  "email": "admin@example.com",
  "password": "Test@1234"
}
```

**Example cURL Command:**

```bash
curl -X POST http://localhost:5000/login \
-H "Content-Type: application/json" \
-d '{
  "email": "admin@example.com",
  "password": "Test@1234"
}'
```

**Response:**

```json
{
  "token": "your-access-token",
  "name": "Test User"
}
```

---

### 3. **Get All Sessions**

**Endpoint:** `GET /sessions`

**Description:**  
Retrieves all chat sessions for the authenticated user.

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X GET http://localhost:5000/sessions \
-H "Authorization: Bearer your-access-token"
```

**Response:**

```json
[
  {
    "id": 1,
    "name": "Chat Session 1",
    "created_at": "2023-10-05T12:00:00Z"
  },
  {
    "id": 2,
    "name": "Chat Session 2",
    "created_at": "2023-10-06T15:30:00Z"
  }
]
```

---

### 4. **Create a New Session**

**Endpoint:** `POST /session`

**Description:**  
Creates a new chat session for the user.

**Request JSON:**

```json
{
  "name": "New Chat Session"
}
```

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X POST http://localhost:5000/session \
-H "Content-Type: application/json" \
-H "Authorization: Bearer your-access-token" \
-d '{
  "name": "New Chat Session"
}'
```

**Response:**

```json
{
  "msg": "Session created",
  "session_id": 3
}
```

---

### 5. **Delete a Session**

**Endpoint:** `DELETE /session/<session_id>`

**Description:**  
Deletes a specific chat session and all associated chats.

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X DELETE http://localhost:5000/session/1 \
-H "Authorization: Bearer your-access-token"
```

**Response:**

```json
{
  "msg": "Session deleted successfully"
}
```

---

### 6. **Rename a Session**

**Endpoint:** `PUT /session/<session_id>/rename`

**Description:**  
Renames an existing chat session.

**Request JSON:**

```json
{
  "name": "Updated Session Name"
}
```

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X PUT http://localhost:5000/session/1/rename \
-H "Content-Type: application/json" \
-H "Authorization: Bearer your-access-token" \
-d '{
  "name": "Updated Session Name"
}'
```

**Response:**

```json
{
  "msg": "Session renamed successfully",
  "new_name": "Updated Session Name"
}
```

---

### 7. **Send a Chat Message**

**Endpoint:** `POST /chat`

**Description:**  
Sends a message to the AI and retrieves a response. If no session ID is provided, a new session is created automatically. The system stores embeddings of the conversation using Pinecone for context retrieval in future interactions.

**Request JSON:**

```json
{
  "message": "Hello, AI!",
  "session_id": 1
}
```

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X POST http://localhost:5000/chat \
-H "Content-Type: application/json" \
-H "Authorization: Bearer your-access-token" \
-d '{
  "message": "Hello, AI!",
  "session_id": 1
}'
```

**Response:**

```json
{
  "response": "Hello! How can I assist you today?",
  "session_id": 1
}
```

**Implementation Details:**

- The message is first sent to Pinecone to retrieve relevant context from previous conversations
- The conversation context and user message are sent to OpenAI using the GPT-4o-mini model
- The user message and AI response are stored in the database
- For messages longer than 10 characters, the system creates an embedding and stores it in Pinecone for future context retrieval

---

### 8. **Get Chat History**

**Endpoint:** `GET /sessions/<session_id>/history`

**Description:**  
Retrieves the chat history for a specific session.

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X GET http://localhost:5000/sessions/1/history \
-H "Authorization: Bearer your-access-token"
```

**Response:**

```json
[
  {
    "message": "Hello, AI!",
    "response": "Hello! How can I assist you today?",
    "timestamp": "2023-10-05T12:05:00Z"
  },
  {
    "message": "Tell me about machine learning",
    "response": "Machine learning is a subfield of artificial intelligence...",
    "timestamp": "2023-10-05T12:07:30Z"
  }
]
```

---

### 9. **Get All Chats for a Session**

**Endpoint:** `GET /chats/<session_id>`

**Description:**  
Retrieves all chats for a specific session, including chat IDs for individual management.

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X GET http://localhost:5000/chats/1 \
-H "Authorization: Bearer your-access-token"
```

**Response:**

```json
[
  {
    "id": 1,
    "message": "Hello, AI!",
    "response": "Hello! How can I assist you today?",
    "timestamp": "2023-10-05T12:05:00Z"
  },
  {
    "id": 2,
    "message": "Tell me about machine learning",
    "response": "Machine learning is a subfield of artificial intelligence...",
    "timestamp": "2023-10-05T12:07:30Z"
  }
]
```

---

### 10. **Delete a Specific Chat**

**Endpoint:** `DELETE /chats/<chat_id>`

**Description:**  
Deletes a specific chat message and its response.

**Headers:**

```json
{
  "Authorization": "Bearer your-access-token"
}
```

**Example cURL Command:**

```bash
curl -X DELETE http://localhost:5000/chats/1 \
-H "Authorization: Bearer your-access-token"
```

**Response:**

```json
{
  "msg": "Chat deleted successfully"
}
```

---

### 11. **AI Handshake (Socket Connection)**

**Endpoint:** `POST /v1/handshake`

**Description:**  
Initializes an AI session, creates memory spaces in Firestore, and returns session metadata including a UUID. This is the starting point for establishing communication with an AI agent.

**Request JSON:**

```json
{
  "ai_name": "Assistant AI"
}
```

**Example cURL Command:**

```bash
curl -X POST http://localhost:5000/v1/handshake \
-H "Content-Type: application/json" \
-d '{
  "ai_name": "Assistant AI"
}'
```

**Response:**

```json
{
  "message": "Handshake successful",
  "sessionData": {
    "ai_name": "Assistant AI",
    "ai_uuid": "123e4567-e89b-12d3-a456-426614174000",
    "created_at": "2023-10-05T12:00:00Z",
    "session_id": 1,
    "internet_access": "safe_only",
    "tools": [],
    "notes": ""
  },
  "dbPaths": {
    "working_memory": "working_memory_123e4567-e89b-12d3-a456-426614174000_Assistant_AI",
    "personal_logs": "personal_logs_123e4567-e89b-12d3-a456-426614174000_Assistant_AI",
    "long_term_memory": "long_term_memory_123e4567-e89b-12d3-a456-426614174000_Assistant_AI",
    "archive": "archive_123e4567-e89b-12d3-a456-426614174000_Assistant_AI"
  }
}
```

**Socket Implementation Details:**

- Upon handshake completion, the server generates a unique UUID for the AI
- The system creates a corresponding socket room using this UUID via `join_room(ai_uuid)`
- Clients connect to the socket server and join their specific room using the `join` event
- The socket implementation uses Flask-SocketIO with threading mode for asynchronous communication
- Each AI instance has its own isolated socket room identified by its UUID
- This enables real-time communication between the client and AI without affecting other connections

---

### 12. **Chat Completions (AI Conversation)**

**Endpoint:** `POST /v1/chat/completions`

**Description:**  
Sends a series of messages to the AI and retrieves a response. The system retrieves context from the AI's working memory before generating a response.

**Request JSON:**

```json
{
  "ai_uuid": "123e4567-e89b-12d3-a456-426614174000",
  "ai_name": "Assistant AI",
  "messages": [{ "role": "user", "content": "What is the weather today?" }],
  "temperature": 0.7,
  "stream": false
}
```

**Example cURL Command:**

```bash
curl -X POST http://localhost:5000/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
  "ai_uuid": "123e4567-e89b-12d3-a456-426614174000",
  "ai_name": "Assistant AI",
  "messages": [
    {"role": "user", "content": "What is the weather today?"}
  ]
}'
```

**Response (Non-streaming):**

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1696507200,
  "model": "gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "I don't have the ability to check the current weather. To get the current weather, you could check a weather website or app, or ask a virtual assistant that has internet access."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 0,
    "completion_tokens": 0,
    "total_tokens": 0
  }
}
```

**Streaming Response:**

When `stream` is set to `true`, the API returns chunks of the response as they become available.

**Memory Implementation:**

- The system maintains separate memory collections in Firestore for each AI instance
- When a message is received, recent messages from the AI's working memory are fetched
- These messages provide context for the current conversation
- After generating a response, the new message and response are stored in the working memory
- This creates a persistent memory that allows the AI to reference previous interactions

---

## Notes

- Replace `http://localhost:5000` with the actual base URL of the API.
- Replace `your-access-token` with the token obtained from the login step.
- Ensure all required fields are provided in the request JSON for each endpoint.
- The chat functionality integrates with OpenAI's GPT model and uses Pinecone for vector embeddings storage and retrieval.
