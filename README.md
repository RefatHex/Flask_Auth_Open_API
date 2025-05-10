# AI Memory Management System

A sophisticated system for managing multiple AI instances, each with their own unique identity and persistent memory, built entirely on Firestore.

## System Architecture

This system is designed to support a single user (administrator) who can manage multiple AI instances. Each AI has:

- Unique UUID for persistent identity
- Dedicated Firestore collections for memory storage
- Personalized long-term context retention

### Core Components

#### 1. Memory Management (Memory_Weaver.py)

The `MemoryWeaver` class provides a comprehensive memory management system for AI instances:

- **Memory Types:**

  - `working_memory`: Recent context and conversations (short-term)
  - `long_term_memory`: Important information to be retained indefinitely
  - `personal_logs`: AI's internal thoughts and reflections
  - `archive`: Historical records that may be occasionally accessed

- **Key Functions:**
  - `create_ai_memory_space(ai_uuid, ai_name)`: Initializes all memory collections for a new AI
  - `store_memory(ai_uuid, ai_name, memory_type, content, metadata)`: Adds new memories to specified collection
  - `retrieve_memories(ai_uuid, ai_name, memory_type, limit, filter_query)`: Gets memories with optional filtering
  - `move_memory(ai_uuid, ai_name, memory_id, from_type, to_type)`: Relocates memories between collections

#### 2. Data Storage (Firestore)

All data is stored in Firestore with the following collections:

- **ai_registry**: Stores metadata about each AI instance

  - ai_uuid, name, creation date, access permissions

- **chat_sessions**: Individual conversation sessions

  - id, name, user_id, ai_uuid, created_at

- **chats**: All chat messages

  - message, response, user_id, session_id, timestamp

- **memory collections**: Specialized collections for each AI and memory type
  - Format: `{memory_type}_{ai_uuid}_{ai_name}`
  - Example: `working_memory_1234_assistant`

#### 3. Socket Communication (sockets.py)

Real-time communication framework to provide immediate updates and interactions:

- Event handlers for various socket events
- Authentication system to verify connections
- Room-based messaging for targeted AI communications
- Memory update notifications

#### 4. Socket Clients (ai_socket_client.py)

Client-side socket connection utilities for AI instances:

- Authentication with server
- Room joining based on AI UUID
- Memory update notifications
- Heartbeat/keep-alive functionality

## API Endpoints

### AI Management

- **POST /v1/handshake**

  - Creates a new AI instance with unique UUID
  - Initializes memory collections
  - Returns session data and database paths
  - Example payload: `{"ai_name": "Assistant Name"}`

- **POST /v1/chat/completions**
  - Processes new messages for an AI
  - Retrieves relevant memory context
  - Gets completion from language model
  - Stores response in memory

### Memory Management

- **POST /create_memory**

  - Creates a new memory entry in specified collection
  - Required fields: `ai_name`, `ai_uuid`, `memory_type`, `memory_data`

- **POST /edit_memory**

  - Updates an existing memory entry
  - Required fields: `ai_name`, `ai_uuid`, `memory_type`, `memory_id`, `memory_data`

- **POST /delete_memory**

  - Removes a memory entry
  - Required fields: `ai_name`, `ai_uuid`, `memory_type`, `memory_id`

- **POST /move_memory**
  - Relocates memory between collections
  - Required fields: `ai_name`, `ai_uuid`, `from_memory_type`, `to_memory_type`, `document_id`

## Data Flow

1. **AI Creation:**

   - Client calls `/v1/handshake` with AI name
   - System creates UUID, initializes memory collections
   - Returns session data with UUID

2. **Memory Storage:**

   - AI interactions stored in Firestore collections
   - Organized by memory type (working, long-term, etc.)
   - UUID-based collection names ensure isolation

3. **Conversation Context:**

   - System retrieves recent memories for context
   - Combines with current messages
   - Sends to language model for processing
   - Stores response in working memory

4. **Memory Management:**
   - Important information can be moved between collections
   - Long-term memory persists across conversations
   - Archives can store historical context

## Socket Events

- **connect**: Initial connection to server
- **authenticate**: Verify connection with API key
- **join**: Join an AI-specific room by UUID
- **memory_update**: Notify about memory changes
- **message**: General communication channel

## Deployment Guide

### Prerequisites

- Python 3.9+
- Firebase account with Firestore database
- OpenAI API key

### Setup

1. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

2. Configure environment variables in `.env` file:

   - `FLASK_APP_SECRET_KEY`: Application security key
   - `JWT_SECRET_KEY`: JWT token key
   - `OPENAI_API_KEY_FLASK_APP`: OpenAI API key
   - `ADMIN_EMAIL`: Administrator email

3. Place Firebase service account key in `firebase_sa.json`

4. Start the server:
   ```bash
   python App.py
   ```

## Security Considerations

- All API calls require authentication
- Socket connections must be authenticated
- AI UUIDs are used for permission scoping
- Access controls defined in AI registry

## Extension Points

The system is designed to be extended with:

1. **AI Capability System**: Add new tools and functions to specific AI instances
2. **Permission System**: Fine-grained access control for AI capabilities
3. **Multi-user Support**: Scale to support multiple users, each with their own AI instances
4. **Enhanced Memory Processing**: Advanced memory retrieval and summarization
5. **Custom Tools**: Integration with external APIs and services
