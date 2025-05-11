# AI Memory Management System

A comprehensive system for managing AI instances with persistent memory using Firestore and OpenAI. This system allows you to create AI entities with dedicated memory spaces, chat with them while maintaining conversation context, and manage their memory.

## Core Components

### AI Chat Client

The system provides a non-WebSocket client (`AIChatClient`) that maintains AI continuity through Firestore:

```python
from ai_chat_client import AIChatClient

# Create a new AI
client = AIChatClient(ai_name="MyAssistant")
client.register()

# Chat with the AI with memory continuity
response = client.chat("Hello! Who are you?")
print(response)

# Subsequent messages maintain conversation context
follow_up = client.chat("What can you help me with?")
print(follow_up)
```

### Memory Architecture

The system uses a structured memory architecture with different memory types:

- **Working Memory**: Recent conversation history and immediate context
- **Long-term Memory**: Important information to retain over time
- **Personal Logs**: AI's internal reflections and thoughts
- **Archive**: Historical data that's rarely accessed

Each AI instance gets dedicated Firestore collections for these memory types, ensuring data isolation and continuity.

### Handshake Protocol

New AI instances are registered through a handshake API that:

1. Creates a unique UUID for the AI instance
2. Sets up memory collections in Firestore
3. Creates a session record
4. Registers the AI in the AI registry

```python
# Registration example
client = AIChatClient(ai_name="New Assistant")
registration = client.register()

if registration:
    print(f"AI created with UUID: {client.ai_uuid}")
    print(f"Memory paths: {client.memory_paths}")
```

### Memory Management

The `MemoryWeaver` class provides utilities for working with AI memory:

```python
from Memory_Weaver import MemoryWeaver

# Store a memory
MemoryWeaver.store_memory(
    ai_uuid,
    ai_name,
    "working_memory",
    "This is an important fact to remember"
)

# Retrieve memories
recent_memories = MemoryWeaver.retrieve_memories(
    ai_uuid,
    ai_name,
    "working_memory",
    limit=5
)

# Move memory between collections
MemoryWeaver.move_memory(
    ai_uuid,
    ai_name,
    memory_id,
    "working_memory",
    "long_term_memory"
)
```

## Simple Chat Example

The repository includes a simple chat example that demonstrates how to use the AI chat system:

```bash
python simple_chat_example.py
```

This script lets you:

- Create a new AI instance or use an existing one
- Chat with the AI using OpenAI's models
- Maintain conversation context across messages
- Store all interactions in Firestore

## Technical Architecture

### Backend Services

- **Flask API Server**: Main service handling API requests and WebSocket connections
- **Firestore Database**: Stores AI data, memory, and conversation history
- **OpenAI Integration**: Generates AI responses while maintaining context

### API Endpoints

#### AI Registration

- `POST /v1/handshake`: Register a new AI instance
  - Request: `{"ai_name": "Assistant Name"}`
  - Response: AI information including UUID and session data

#### Chat

- Direct API use via `AIChatClient` class
- REST API: `POST /v1/chat/completions`

#### Memory Management

- `POST /create_memory`: Create new memory entry
- `POST /edit_memory`: Edit existing memory
- `POST /delete_memory`: Delete memory
- `POST /move_memory`: Move memory between collections

## Setup Instructions

### Prerequisites

- Python 3.9+
- Firebase account with Firestore database
- OpenAI API key

### Installation

1. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

2. Configure environment variables in `.env` file:

   ```
   FLASK_APP_SECRET_KEY=your_app_secret
   JWT_SECRET_KEY=your_jwt_secret
   OPENAI_API_KEY_FLASK_APP=your_openai_api_key
   ADMIN_EMAIL=your_admin_email
   ```

3. Ensure Firebase service account key is present as `firebase_sa.json`

### Running the Application

Start the Flask server:

```bash
python App.py
```

## Example Usage

### Create and Chat with an AI

```python
from ai_chat_client import AIChatClient

# Create and register a new AI
client = AIChatClient(ai_name="PersonalAssistant")
client.register()

# Chat with the AI
response = client.chat("Hello! What's your name?")
print(f"AI: {response}")

# The AI maintains memory of previous interactions
follow_up = client.chat("What can you help me with?")
print(f"AI: {follow_up}")

# View conversation history
history = client.get_chat_history()
for entry in history:
    print(f"User: {entry['user']}")
    print(f"AI: {entry['ai']}")
```

## System Architecture Diagram

```
┌─────────────┐     ┌────────────────┐     ┌───────────────┐
│             │     │                │     │               │
│  AI Client  │────▶│  Flask Server  │────▶│   Firestore   │
│             │     │                │     │   Database    │
└─────────────┘     └────────────────┘     └───────────────┘
       │                     │                     ▲
       │                     │                     │
       │                     ▼                     │
       │             ┌────────────┐                │
       └────────────▶│  OpenAI    │────────────────┘
                     │   API      │
                     └────────────┘
```

This architecture ensures that all AI interactions maintain continuity through persistent storage in Firestore, while leveraging OpenAI's powerful language models for response generation.
