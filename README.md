# AI Chat System with Memory

This system allows you to have continuous conversations with AI assistants that remember previous interactions. The project uses Firebase for storing conversation history, OpenAI for generating responses, and Flask for the server.

## Main Components

### 1. AI Registration (Handshake)

When you create a new AI assistant:

- The system generates a unique ID (UUID) for the AI
- Creates memory spaces in Firestore to store conversations
- Sets up a chat session
- Returns the AI's UUID which is needed for future conversations

Example:

```python
# Create a new AI assistant
from ai_chat_client import AIChatClient

# Create and register a new AI with a name
client = AIChatClient(ai_name="MyAssistant")
registration = client.register()

# The AI is now registered and has a UUID
print(f"AI UUID: {client.ai_uuid}")
```

### 2. Chatting with Memory

The most important feature of this system is that AI assistants remember previous conversations:

- Each message and response is stored in the AI's "working memory"
- When you chat with the AI, it retrieves relevant context from past conversations
- This allows for continuous, coherent conversations over time

Example:

```python
# Chat with your AI and get responses that remember context
from ai_chat_client import AIChatClient

# Create client with a registered AI
client = AIChatClient(ai_name="MyAssistant")
client.register()  # Get a new UUID and memory space

# First message
response1 = client.chat("Hello! My name is Angela.")
print(response1)  # The AI will introduce itself

# Second message - the AI will remember your name
response2 = client.chat("What's my name?")
print(response2)  # The AI will recall that your name is Angela
```

### 3. How Memory Works

The system uses Firestore collections to store memory for each AI:

- **Working Memory**: Stores recent conversations for immediate context
- **Long-term Memory**: For important information that should be remembered
- **Archive**: For storing older conversations
- **Personal Logs**: For AI's personal reflections

All conversations are automatically saved in working memory and used for context in future chats.

## Key Files

- `App.py`: The main server with API endpoints for chat and AI registration
- `ai_chat_client.py`: Client for easy interaction with AI assistants
- `Memory_Weaver.py`: Manages different types of AI memory
- `models.py`: Handles database interactions with Firestore

## Simple Usage

```python
from ai_chat_client import AIChatClient

# Create a new AI assistant
client = AIChatClient(ai_name="PersonalAssistant")
registration = client.register()

if registration:
    # Have a conversation
    client.chat("Hi there, I'm new here!")
    client.chat("Can you remember things I tell you?")
    client.chat("My favorite color is blue.")

    # Later, even in a new session, with the same AI UUID:
    client.chat("What's my favorite color?")  # The AI will remember it's blue
```

The key to continuous conversation is using the same AI UUID across sessions. You can save the UUID after registration and use it later:

```python
# Save this UUID for later use with the same AI
ai_uuid = client.ai_uuid

# Later, reconnect using the saved UUID
reconnected_client = AIChatClient(ai_name="PersonalAssistant", ai_uuid=ai_uuid)
reconnected_client.chat("Do you remember our previous conversation?")  # It will!
```

## Working with the AI Chat Client

The `AIChatClient` class provides a simple interface for chatting with AI assistants:

1. Create a client with an AI name
2. Register to get a UUID (or provide an existing UUID)
3. Use the `chat()` method to have conversations

That's it! The system handles all the memory management automatically.

## API Documentation

### Base URL

All API endpoints are available at the base URL: `http://localhost:5000` or `http://127.0.0.1:5000`

If accessing the server from another computer on the network, use the server's IP address: `http://<server-ip-address>:5000`

### Authentication APIs

#### 1. User Signup

```
POST /signup
```

Register a new user (restricted to admin).

**Request Body:**

```json
{
  "name": "Admin User",
  "email": "admin@example.com",
  "password": "securepassword123",
  "confirm": "securepassword123"
}
```

**Response (201 Created):**

```json
{
  "msg": "User created successfully"
}
```

#### 2. User Login

```
POST /login
```

Log in and get a JWT token.

**Request Body:**

```json
{
  "email": "admin@example.com",
  "password": "securepassword123"
}
```

**Response (200 OK):**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "name": "Admin User"
}
```

### Chat APIs

#### 1. Send Chat Message

```
POST /chat
```

Send a message and get an AI response. Requires JWT authentication.

**Headers:**

```
Authorization: Bearer [your_jwt_token]
```

**Request Body:**

```json
{
  "message": "Hello, how are you today?",
  "session_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

_Note: `session_id` is optional. If not provided, a new session will be created._

**Response (200 OK):**

```json
{
  "response": "I'm doing well, thank you for asking! How can I help you today?",
  "session_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

### Session Management APIs

#### 1. Get All Chat Sessions

```
GET /sessions
```

Get all chat sessions for the authenticated user. Requires JWT authentication.

**Headers:**

```
Authorization: Bearer [your_jwt_token]
```

**Response (200 OK):**

```json
[
  {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "First Chat",
    "user_id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
    "created_at": "2024-06-30T15:30:45.123Z",
    "ai_uuid": "abcdef12-3456-7890-abcd-ef1234567890"
  },
  {
    "id": "7c9a8b7c-6d5e-4f3g-2h1i-0j9k8l7m6n5o",
    "name": "Project Discussion",
    "user_id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
    "created_at": "2024-06-29T10:15:30.789Z"
  }
]
```

#### 2. Create New Chat Session

```
POST /session
```

Create a new chat session. Requires JWT authentication.

**Headers:**

```
Authorization: Bearer [your_jwt_token]
```

**Request Body:**

```json
{
  "name": "Technical Support"
}
```

**Response (201 Created):**

```json
{
  "msg": "Session created",
  "session_id": "5f6g7h8i-9j0k-1l2m-3n4o-5p6q7r8s9t0u"
}
```

#### 3. Delete Chat Session

```
DELETE /session/{session_id}
```

Delete a specific chat session and its messages. Requires JWT authentication.

**Headers:**

```
Authorization: Bearer [your_jwt_token]
```

**Response (200 OK):**

```json
{
  "msg": "Session deleted successfully"
}
```

#### 4. Rename Chat Session

```
PUT /session/{session_id}/rename
```

Rename an existing chat session. Requires JWT authentication.

**Headers:**

```
Authorization: Bearer [your_jwt_token]
```

**Request Body:**

```json
{
  "name": "Updated Session Name"
}
```

**Response (200 OK):**

```json
{
  "msg": "Session renamed successfully",
  "new_name": "Updated Session Name"
}
```

### AI Management APIs

#### 1. AI Handshake (Registration)

```
POST /v1/handshake
```

Register a new AI assistant and create memory spaces.

**Request Body:**

```json
{
  "ai_name": "Assistant"
}
```

**Response (200 OK):**

```json
{
  "message": "Handshake successful",
  "sessionData": {
    "ai_name": "Assistant",
    "ai_uuid": "6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a",
    "created_at": "2024-06-30T12:34:56.789Z",
    "session_id": "a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6",
    "internet_access": "safe_only",
    "tools": []
  },
  "dbPaths": {
    "working_memory": "working_memory_6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a_Assistant",
    "personal_logs": "personal_logs_6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a_Assistant",
    "long_term_memory": "long_term_memory_6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a_Assistant",
    "archive": "archive_6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a_Assistant"
  }
}
```

#### 2. AI Chat Completions

```
POST /v1/chat/completions
```

Get AI response with memory context.

**Request Body:**

```json
{
  "ai_uuid": "6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a",
  "ai_name": "Assistant",
  "messages": [
    {
      "role": "user",
      "content": "What's the weather like today?"
    }
  ],
  "temperature": 0.7,
  "stream": false
}
```

**Response (200 OK):**

```json
{
  "id": "chatcmpl-123abc",
  "object": "chat.completion",
  "created": 1719766545,
  "model": "gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "I don't have access to real-time weather data or your location. To get accurate weather information, you can check a weather website like weather.com, use a weather app on your device, or simply look outside. If you'd like me to discuss weather in general or answer other questions, I'd be happy to help!"
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

### Memory Management APIs

#### 1. Create Memory

```
POST /create_memory
```

Create a new memory entry in an AI's memory.

**Request Body:**

```json
{
  "ai_name": "Assistant",
  "ai_uuid": "6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a",
  "memory_type": "working_memory",
  "memory_data": {
    "content": "User's favorite color is blue",
    "importance": "high"
  }
}
```

**Response (200 OK):**

```json
{
  "status": "Memory created successfully."
}
```

#### 2. Edit Memory

```
POST /edit_memory
```

Update an existing memory entry.

**Request Body:**

```json
{
  "ai_name": "Assistant",
  "ai_uuid": "6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a",
  "memory_type": "working_memory",
  "memory_id": "b2c3d4e5-f6g7-h8i9-j0k1-l2m3n4o5p6q7",
  "memory_data": {
    "content": "User's favorite color is green",
    "importance": "high"
  }
}
```

**Response (200 OK):**

```json
{
  "status": "Memory updated successfully."
}
```

#### 3. Delete Memory

```
POST /delete_memory
```

Delete a memory entry.

**Request Body:**

```json
{
  "ai_name": "Assistant",
  "ai_uuid": "6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a",
  "memory_type": "working_memory",
  "memory_id": "b2c3d4e5-f6g7-h8i9-j0k1-l2m3n4o5p6q7"
}
```

**Response (200 OK):**

```json
{
  "status": "Memory deleted successfully."
}
```

#### 4. Move Memory

```
POST /move_memory
```

Move a memory entry from one collection to another.

**Request Body:**

```json
{
  "ai_name": "Assistant",
  "ai_uuid": "6d3a8f5e-2b1c-4d7f-9e6a-0c1b2d3e4f5a",
  "from_memory_type": "working_memory",
  "to_memory_type": "long_term_memory",
  "document_id": "b2c3d4e5-f6g7-h8i9-j0k1-l2m3n4o5p6q7"
}
```

**Response (200 OK):**

```json
{
  "status": "Memory moved successfully."
}
```
