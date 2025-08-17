# Database Schema Documentation

## Overview

The Chat App uses MongoDB as the primary database with three main collections:
- **Users** - User account information
- **Chats** - Chat room/conversation metadata
- **Messages** - Individual messages within chats

## Database Models

### User Model

**Collection:** `users`

```typescript
interface IUser {
  _id: ObjectId;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
}
```

**Schema Definition:**
```javascript
{
  name: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: true,
    unique: true
  }
}
```

**Field Descriptions:**
- `_id` - MongoDB ObjectId, auto-generated primary key
- `name` - User's display name (initially derived from email prefix)
- `email` - User's email address (unique identifier for authentication)
- `createdAt` - Timestamp when user account was created
- `updatedAt` - Timestamp when user account was last modified

**Indexes:**
- `email` - Unique index for fast email lookups during authentication

---

### Chat Model

**Collection:** `chats`

```typescript
interface IChat {
  _id: ObjectId;
  users: string[];
  latestMessage: {
    text: string;
    sender: string;
  };
  createdAt: Date;
  updatedAt: Date;
}
```

**Schema Definition:**
```javascript
{
  users: [{
    type: String,
    required: true
  }],
  latestMessage: {
    text: String,
    sender: String
  }
}
```

**Field Descriptions:**
- `_id` - MongoDB ObjectId, auto-generated primary key
- `users` - Array of user IDs participating in the chat (currently supports 2 users)
- `latestMessage` - Object containing the most recent message preview
  - `text` - Preview text of the latest message
  - `sender` - User ID of the message sender
- `createdAt` - Timestamp when chat was created
- `updatedAt` - Timestamp when chat was last updated (usually when new message sent)

**Indexes:**
- `users` - Index for efficient chat lookups by participant
- `updatedAt` - Index for sorting chats by most recent activity

---

### Message Model

**Collection:** `messages`

```typescript
interface IMessage {
  _id: ObjectId;
  chatId: ObjectId;
  sender: string;
  text?: string;
  image?: {
    url: string;
    publicId: string;
  };
  messageType: "text" | "image";
  seen: boolean;
  seenAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}
```

**Schema Definition:**
```javascript
{
  chatId: {
    type: Schema.Types.ObjectId,
    ref: "Chat",
    required: true
  },
  sender: {
    type: String,
    required: true
  },
  text: String,
  image: {
    url: String,
    publicId: String
  },
  messageType: {
    type: String,
    enum: ["text", "image"],
    default: "text"
  },
  seen: {
    type: Boolean,
    default: false
  },
  seenAt: {
    type: Date,
    default: null
  }
}
```

**Field Descriptions:**
- `_id` - MongoDB ObjectId, auto-generated primary key
- `chatId` - Reference to the Chat document this message belongs to
- `sender` - User ID of the message sender
- `text` - Message text content (optional for image messages)
- `image` - Object containing image information (optional)
  - `url` - Cloudinary URL of the uploaded image
  - `publicId` - Cloudinary public ID for image management
- `messageType` - Type of message: "text" or "image"
- `seen` - Boolean indicating if message has been seen by recipient
- `seenAt` - Timestamp when message was marked as seen
- `createdAt` - Timestamp when message was sent
- `updatedAt` - Timestamp when message was last modified

**Indexes:**
- `chatId` - Index for efficient message retrieval by chat
- `createdAt` - Index for chronological message ordering
- `sender, seen` - Compound index for unseen message counts

---

## Relationships

### User ↔ Chat
- **Type:** Many-to-Many
- **Implementation:** Chat document contains array of user IDs
- **Current Limitation:** Only supports 2-user chats (direct messages)

### Chat ↔ Message
- **Type:** One-to-Many
- **Implementation:** Message document references Chat via `chatId`
- **Cascade:** Messages are not automatically deleted when chat is deleted

### User ↔ Message
- **Type:** One-to-Many (as sender)
- **Implementation:** Message document contains sender user ID as string
- **Note:** User information is fetched via API calls between services

---

## Data Flow

### User Registration/Login
1. User enters email → User Service
2. OTP generated and stored in Redis (5-minute expiry)
3. OTP sent via RabbitMQ to Mail Service
4. User verifies OTP → User document created/retrieved
5. JWT token generated and returned

### Chat Creation
1. User requests chat with another user → Chat Service
2. Check if chat already exists between users
3. If not, create new Chat document with both user IDs
4. Return chat ID for messaging

### Message Sending
1. User sends message → Chat Service
2. Create Message document with chat reference
3. Update Chat document's `latestMessage` field
4. Emit real-time events via Socket.IO
5. Mark as seen if recipient is in chat room

### Message Reading
1. User opens chat → Chat Service
2. Fetch all messages for chat ID
3. Mark unread messages as seen
4. Emit "messagesSeen" event to sender
5. Return messages with user information

---

## Redis Schema

### OTP Storage
**Key Pattern:** `otp:{email}`
**Value:** 6-digit OTP string
**TTL:** 300 seconds (5 minutes)

**Example:**
```
Key: otp:user@example.com
Value: "123456"
TTL: 300
```

### Rate Limiting
**Key Pattern:** `otp:ratelimit:{email}`
**Value:** "true"
**TTL:** 60 seconds (1 minute)

**Example:**
```
Key: otp:ratelimit:user@example.com
Value: "true"
TTL: 60
```

---

## Performance Considerations

### Indexing Strategy
- **Users:** Email unique index for authentication
- **Chats:** Users array index for participant lookups
- **Messages:** Compound indexes for chat queries and unseen counts

### Query Optimization
- Chat list queries sort by `updatedAt` for recent activity
- Message queries use `chatId` index with chronological sorting
- Unseen message counts use compound index on `sender` and `seen`

### Scalability Notes
- Current design supports direct messaging (2 users per chat)
- For group chats, `users` array would need size considerations
- Message history could be partitioned by date for large volumes
- Consider read replicas for message retrieval operations

---

## Data Validation

### User Validation
- Email format validation
- Unique email constraint
- Name length limits (configurable)

### Chat Validation
- Exactly 2 users required for current implementation
- User existence validation via User Service API

### Message Validation
- Either text or image required
- Chat participation validation
- File type and size validation for images
- Message type consistency (text/image)

---

## Migration Considerations

### Schema Evolution
- Add fields with default values for backward compatibility
- Use MongoDB's flexible schema for gradual migrations
- Consider versioning for major schema changes

### Data Cleanup
- Implement soft deletes for user accounts
- Archive old messages based on retention policies
- Clean up orphaned Cloudinary images
- Remove expired Redis keys automatically
