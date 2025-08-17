# API Documentation

## Overview

This document provides comprehensive API documentation for the Chat App microservices architecture. The application consists of three main services:

- **User Service** - Authentication and user management
- **Chat Service** - Real-time messaging and chat management  
- **Mail Service** - Email notifications and OTP handling

## Base URLs

- User Service: `http://localhost:3001/api/v1`
- Chat Service: `http://localhost:3002/api/v1`
- Socket.IO: `http://localhost:3002`

## Authentication

Most endpoints require JWT authentication. Include the token in the Authorization header:

```
Authorization: Bearer <jwt_token>
```

---

## User Service API

### POST /login
Initiate user login by sending OTP to email.

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response (200):**
```json
{
  "message": "OTP sent to your mail"
}
```

**Response (429 - Rate Limited):**
```json
{
  "message": "Too many requests. Please wait before requesting new otp"
}
```

### POST /verify
Verify OTP and complete authentication.

**Request Body:**
```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

**Response (200):**
```json
{
  "message": "User Verified",
  "user": {
    "_id": "64f1a2b3c4d5e6f7g8h9i0j1",
    "name": "user@exa",
    "email": "user@example.com",
    "createdAt": "2023-09-01T10:00:00.000Z",
    "updatedAt": "2023-09-01T10:00:00.000Z"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (400):**
```json
{
  "message": "Invalid or expired OTP"
}
```

### GET /me
Get current user profile (requires authentication).

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
{
  "_id": "64f1a2b3c4d5e6f7g8h9i0j1",
  "name": "John Doe",
  "email": "john@example.com",
  "createdAt": "2023-09-01T10:00:00.000Z",
  "updatedAt": "2023-09-01T10:00:00.000Z"
}
```

### GET /user/all
Get all users (requires authentication).

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
[
  {
    "_id": "64f1a2b3c4d5e6f7g8h9i0j1",
    "name": "John Doe",
    "email": "john@example.com",
    "createdAt": "2023-09-01T10:00:00.000Z",
    "updatedAt": "2023-09-01T10:00:00.000Z"
  },
  {
    "_id": "64f1a2b3c4d5e6f7g8h9i0j2",
    "name": "Jane Smith",
    "email": "jane@example.com",
    "createdAt": "2023-09-01T11:00:00.000Z",
    "updatedAt": "2023-09-01T11:00:00.000Z"
  }
]
```

### GET /user/:id
Get specific user by ID.

**Parameters:**
- `id` (string): User ID

**Response (200):**
```json
{
  "_id": "64f1a2b3c4d5e6f7g8h9i0j1",
  "name": "John Doe",
  "email": "john@example.com",
  "createdAt": "2023-09-01T10:00:00.000Z",
  "updatedAt": "2023-09-01T10:00:00.000Z"
}
```

### POST /update/user
Update user profile (requires authentication).

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "name": "Updated Name"
}
```

**Response (200):**
```json
{
  "message": "User Updated",
  "user": {
    "_id": "64f1a2b3c4d5e6f7g8h9i0j1",
    "name": "Updated Name",
    "email": "user@example.com",
    "createdAt": "2023-09-01T10:00:00.000Z",
    "updatedAt": "2023-09-01T12:00:00.000Z"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## Chat Service API

### POST /chat/new
Create a new chat between two users (requires authentication).

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "otherUserId": "64f1a2b3c4d5e6f7g8h9i0j2"
}
```

**Response (201):**
```json
{
  "message": "New Chat created",
  "chatId": "64f1a2b3c4d5e6f7g8h9i0j3"
}
```

**Response (200 - Chat exists):**
```json
{
  "message": "Chat already exists",
  "chatId": "64f1a2b3c4d5e6f7g8h9i0j3"
}
```

### GET /chat/all
Get all chats for the authenticated user (requires authentication).

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200):**
```json
{
  "chats": [
    {
      "user": {
        "_id": "64f1a2b3c4d5e6f7g8h9i0j2",
        "name": "Jane Smith",
        "email": "jane@example.com"
      },
      "chat": {
        "_id": "64f1a2b3c4d5e6f7g8h9i0j3",
        "users": ["64f1a2b3c4d5e6f7g8h9i0j1", "64f1a2b3c4d5e6f7g8h9i0j2"],
        "latestMessage": {
          "text": "Hello there!",
          "sender": "64f1a2b3c4d5e6f7g8h9i0j2"
        },
        "unseenCount": 2,
        "createdAt": "2023-09-01T10:00:00.000Z",
        "updatedAt": "2023-09-01T12:00:00.000Z"
      }
    }
  ]
}
```

### POST /message
Send a message (text or image) to a chat (requires authentication).

**Headers:**
```
Authorization: Bearer <jwt_token>
Content-Type: multipart/form-data
```

**Request Body (Form Data):**
- `chatId` (string): Chat ID
- `text` (string, optional): Message text
- `image` (file, optional): Image file

**Response (201):**
```json
{
  "message": {
    "_id": "64f1a2b3c4d5e6f7g8h9i0j4",
    "chatId": "64f1a2b3c4d5e6f7g8h9i0j3",
    "sender": "64f1a2b3c4d5e6f7g8h9i0j1",
    "text": "Hello there!",
    "messageType": "text",
    "seen": false,
    "createdAt": "2023-09-01T12:00:00.000Z",
    "updatedAt": "2023-09-01T12:00:00.000Z"
  },
  "sender": "64f1a2b3c4d5e6f7g8h9i0j1"
}
```

**Response (201 - Image message):**
```json
{
  "message": {
    "_id": "64f1a2b3c4d5e6f7g8h9i0j5",
    "chatId": "64f1a2b3c4d5e6f7g8h9i0j3",
    "sender": "64f1a2b3c4d5e6f7g8h9i0j1",
    "text": "Check this out!",
    "image": {
      "url": "https://res.cloudinary.com/demo/image/upload/v1234567890/sample.jpg",
      "publicId": "sample"
    },
    "messageType": "image",
    "seen": false,
    "createdAt": "2023-09-01T12:00:00.000Z",
    "updatedAt": "2023-09-01T12:00:00.000Z"
  },
  "sender": "64f1a2b3c4d5e6f7g8h9i0j1"
}
```

### GET /message/:chatId
Get all messages for a specific chat (requires authentication).

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Parameters:**
- `chatId` (string): Chat ID

**Response (200):**
```json
{
  "messages": [
    {
      "_id": "64f1a2b3c4d5e6f7g8h9i0j4",
      "chatId": "64f1a2b3c4d5e6f7g8h9i0j3",
      "sender": "64f1a2b3c4d5e6f7g8h9i0j1",
      "text": "Hello there!",
      "messageType": "text",
      "seen": true,
      "seenAt": "2023-09-01T12:01:00.000Z",
      "createdAt": "2023-09-01T12:00:00.000Z",
      "updatedAt": "2023-09-01T12:01:00.000Z"
    },
    {
      "_id": "64f1a2b3c4d5e6f7g8h9i0j5",
      "chatId": "64f1a2b3c4d5e6f7g8h9i0j3",
      "sender": "64f1a2b3c4d5e6f7g8h9i0j2",
      "text": "Hi! How are you?",
      "messageType": "text",
      "seen": false,
      "createdAt": "2023-09-01T12:02:00.000Z",
      "updatedAt": "2023-09-01T12:02:00.000Z"
    }
  ],
  "user": {
    "_id": "64f1a2b3c4d5e6f7g8h9i0j2",
    "name": "Jane Smith",
    "email": "jane@example.com"
  }
}
```

---

## Socket.IO Events

### Client to Server Events

#### `joinChat`
Join a specific chat room.

**Payload:**
```javascript
socket.emit('joinChat', chatId);
```

#### `leaveChat`
Leave a specific chat room.

**Payload:**
```javascript
socket.emit('leaveChat', chatId);
```

#### `typing`
Indicate user is typing in a chat.

**Payload:**
```javascript
socket.emit('typing', {
  chatId: '64f1a2b3c4d5e6f7g8h9i0j3',
  userId: '64f1a2b3c4d5e6f7g8h9i0j1'
});
```

#### `stopTyping`
Indicate user stopped typing in a chat.

**Payload:**
```javascript
socket.emit('stopTyping', {
  chatId: '64f1a2b3c4d5e6f7g8h9i0j3',
  userId: '64f1a2b3c4d5e6f7g8h9i0j1'
});
```

### Server to Client Events

#### `getOnlineUser`
Receive list of online users.

**Payload:**
```javascript
socket.on('getOnlineUser', (onlineUsers) => {
  // onlineUsers is an array of user IDs
  console.log('Online users:', onlineUsers);
});
```

#### `newMessage`
Receive new message in real-time.

**Payload:**
```javascript
socket.on('newMessage', (message) => {
  console.log('New message:', message);
  // message object same as API response
});
```

#### `messagesSeen`
Receive notification when messages are seen.

**Payload:**
```javascript
socket.on('messagesSeen', (data) => {
  console.log('Messages seen:', data);
  // data: { chatId, seenBy, messageIds }
});
```

#### `userTyping`
Receive notification when user is typing.

**Payload:**
```javascript
socket.on('userTyping', (data) => {
  console.log('User typing:', data);
  // data: { chatId, userId }
});
```

#### `userStoppedTyping`
Receive notification when user stopped typing.

**Payload:**
```javascript
socket.on('userStoppedTyping', (data) => {
  console.log('User stopped typing:', data);
  // data: { chatId, userId }
});
```

---

## Error Responses

All endpoints may return the following error responses:

### 400 Bad Request
```json
{
  "message": "Validation error message"
}
```

### 401 Unauthorized
```json
{
  "message": "Unauthorized"
}
```

### 403 Forbidden
```json
{
  "message": "You are not a participant of this chat"
}
```

### 404 Not Found
```json
{
  "message": "Resource not found"
}
```

### 429 Too Many Requests
```json
{
  "message": "Too many requests. Please wait before requesting new otp"
}
```

### 500 Internal Server Error
```json
{
  "message": "Internal server error"
}
```

---

## Rate Limiting

- **OTP Requests**: Limited to 1 request per minute per email address
- **OTP Validity**: 5 minutes from generation

---

## File Upload Specifications

### Image Upload
- **Supported formats**: JPG, JPEG, PNG, GIF, WebP
- **Maximum file size**: 10MB (configurable via Cloudinary)
- **Storage**: Cloudinary cloud storage
- **Processing**: Automatic optimization and format conversion

### Upload Response
Images are automatically uploaded to Cloudinary and return:
```json
{
  "url": "https://res.cloudinary.com/demo/image/upload/v1234567890/sample.jpg",
  "publicId": "sample"
}
```
