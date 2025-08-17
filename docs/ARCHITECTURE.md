# Architecture Documentation

## System Overview

The Chat App follows a **microservices architecture** with clear separation of concerns, enabling scalability, maintainability, and independent deployment of services.

## High-Level Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Load Balancer  │    │   API Gateway   │
│   (Next.js)     │◄──►│   (Optional)     │◄──►│   (Optional)    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                                               │
         ▼                                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Backend Services                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ User Service│  │Chat Service │  │Mail Service │            │
│  │ Port: 3001  │  │ Port: 3002  │  │ Port: 3003  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Infrastructure Layer                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   MongoDB   │  │    Redis    │  │  RabbitMQ   │            │
│  │ (Database)  │  │  (Cache)    │  │(Message Q)  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐                             │
│  │ Cloudinary  │  │  Socket.IO  │                             │
│  │(File Store) │  │(Real-time)  │                             │
│  └─────────────┘  └─────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

## Service Architecture

### 1. User Service
**Responsibilities:**
- User authentication and authorization
- User profile management
- JWT token generation and validation
- OTP generation and validation
- Rate limiting for authentication requests

**Technology Stack:**
- Node.js + Express + TypeScript
- MongoDB (User data storage)
- Redis (OTP storage and rate limiting)
- RabbitMQ (Email queue publishing)
- JWT for authentication

**Key Components:**
```
User Service
├── Controllers/
│   └── user.ts (Authentication logic)
├── Models/
│   └── User.ts (User schema)
├── Middleware/
│   └── isAuth.ts (JWT validation)
├── Config/
│   ├── db.ts (MongoDB connection)
│   ├── rabbitmq.ts (Message queue)
│   └── generateToken.ts (JWT utilities)
└── Routes/
    └── user.ts (API endpoints)
```

### 2. Chat Service
**Responsibilities:**
- Real-time messaging with Socket.IO
- Chat room management
- Message storage and retrieval
- File upload handling
- Message seen/unseen tracking

**Technology Stack:**
- Node.js + Express + TypeScript
- MongoDB (Chat and message storage)
- Socket.IO (Real-time communication)
- Cloudinary (Image storage)
- Multer (File upload handling)

**Key Components:**
```
Chat Service
├── Controllers/
│   └── chat.ts (Messaging logic)
├── Models/
│   ├── Chat.ts (Chat schema)
│   └── Messages.ts (Message schema)
├── Middleware/
│   ├── isAuth.ts (JWT validation)
│   └── multer.ts (File upload)
├── Config/
│   ├── db.ts (MongoDB connection)
│   ├── socket.ts (Socket.IO setup)
│   └── cloudinary.ts (File storage)
└── Routes/
    └── chat.ts (API endpoints)
```

### 3. Mail Service
**Responsibilities:**
- Email sending (OTP notifications)
- RabbitMQ message consumption
- Email template management
- SMTP configuration

**Technology Stack:**
- Node.js + Express + TypeScript
- RabbitMQ (Message queue consumption)
- Nodemailer (Email sending)

**Key Components:**
```
Mail Service
├── consumer.ts (RabbitMQ consumer)
├── emailService.ts (Email logic)
└── index.ts (Service entry point)
```

### 4. Frontend (Next.js)
**Responsibilities:**
- User interface and experience
- Real-time message display
- Authentication flow
- File upload interface
- Socket.IO client management

**Technology Stack:**
- Next.js 15 + React 19
- TypeScript
- Tailwind CSS
- Socket.IO Client
- Axios (HTTP client)

## Data Flow Diagrams

### Authentication Flow
```
┌─────────┐    ┌──────────────┐    ┌─────────┐    ┌──────────────┐
│Frontend │    │ User Service │    │  Redis  │    │ Mail Service │
└────┬────┘    └──────┬───────┘    └────┬────┘    └──────┬───────┘
     │                │                 │                │
     │ 1. POST /login │                 │                │
     │ {email}        │                 │                │
     ├───────────────►│                 │                │
     │                │ 2. Generate OTP │                │
     │                ├────────────────►│                │
     │                │ 3. Store OTP    │                │
     │                │◄────────────────┤                │
     │                │ 4. Publish to   │                │
     │                │    RabbitMQ     │                │
     │                ├─────────────────┼───────────────►│
     │ 5. OTP sent    │                 │ 6. Send Email  │
     │◄───────────────┤                 │                │
     │                │                 │                │
     │ 7. POST /verify│                 │                │
     │ {email, otp}   │                 │                │
     ├───────────────►│ 8. Validate OTP │                │
     │                ├────────────────►│                │
     │                │ 9. OTP valid    │                │
     │                │◄────────────────┤                │
     │ 10. JWT token  │                 │                │
     │◄───────────────┤                 │                │
```

### Message Flow
```
┌─────────┐    ┌──────────────┐    ┌─────────┐    ┌─────────┐
│Frontend │    │ Chat Service │    │ MongoDB │    │Socket.IO│
└────┬────┘    └──────┬───────┘    └────┬────┘    └────┬────┘
     │                │                 │              │
     │ 1. Send Message│                 │              │
     ├───────────────►│                 │              │
     │                │ 2. Save Message │              │
     │                ├────────────────►│              │
     │                │ 3. Update Chat  │              │
     │                ├────────────────►│              │
     │                │ 4. Emit to Room │              │
     │                ├─────────────────┼─────────────►│
     │                │                 │ 5. Broadcast │
     │ 6. Real-time   │                 │              │
     │    Update      │                 │              │
     │◄───────────────┼─────────────────┼──────────────┤
```

## Inter-Service Communication

### Synchronous Communication (HTTP)
- **Frontend ↔ User Service**: Authentication, profile management
- **Frontend ↔ Chat Service**: Messaging, chat management
- **Chat Service ↔ User Service**: User information retrieval

### Asynchronous Communication (RabbitMQ)
- **User Service → Mail Service**: OTP email notifications

### Real-time Communication (Socket.IO)
- **Frontend ↔ Chat Service**: Live messaging, typing indicators, online status

## Security Architecture

### Authentication & Authorization
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   Service   │    │   Resource  │
└──────┬──────┘    └──────┬──────┘    └──────┬──────┘
       │                  │                  │
       │ 1. Request with  │                  │
       │    JWT Token     │                  │
       ├─────────────────►│                  │
       │                  │ 2. Validate JWT  │
       │                  │                  │
       │                  │ 3. Extract User  │
       │                  │    Information   │
       │                  │                  │
       │                  │ 4. Access        │
       │                  │    Resource      │
       │                  ├─────────────────►│
       │ 5. Response      │ 5. Data          │
       │◄─────────────────┤◄─────────────────┤
```

### Security Measures
- **JWT Authentication**: Stateless token-based authentication
- **Rate Limiting**: OTP request limiting (1/minute per email)
- **CORS Configuration**: Cross-origin request protection
- **Input Validation**: Request payload validation
- **File Upload Security**: Type and size restrictions
- **Environment Variables**: Sensitive data protection

## Scalability Considerations

### Horizontal Scaling
```
┌─────────────────────────────────────────────────────────────┐
│                    Load Balancer                            │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│User Service │ │User Service │ │User Service │
│ Instance 1  │ │ Instance 2  │ │ Instance 3  │
└─────────────┘ └─────────────┘ └─────────────┘
        │             │             │
        └─────────────┼─────────────┘
                      │
                      ▼
              ┌─────────────┐
              │   MongoDB   │
              │   Cluster   │
              └─────────────┘
```

### Performance Optimizations
- **Database Indexing**: Optimized queries for users, chats, messages
- **Redis Caching**: Session management and rate limiting
- **CDN Integration**: Cloudinary for image optimization
- **Connection Pooling**: MongoDB connection optimization
- **Socket.IO Clustering**: Multi-instance real-time support

## Deployment Architecture

### Development Environment
```
┌─────────────────────────────────────────────────────────────┐
│                    Local Development                        │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Next.js   │  │   Node.js   │  │   Node.js   │        │
│  │ Frontend    │  │ Services    │  │ Services    │        │
│  │ Port: 3000  │  │ Port: 3001+ │  │ Port: 3002+ │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   MongoDB   │  │    Redis    │  │  RabbitMQ   │        │
│  │ Port: 27017 │  │ Port: 6379  │  │ Port: 5672  │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### Production Environment (Recommended)
```
┌─────────────────────────────────────────────────────────────┐
│                      Cloud Provider                         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   CDN       │  │Load Balancer│  │   API       │        │
│  │(Cloudflare) │  │   (Nginx)   │  │  Gateway    │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Frontend   │  │  Container  │  │  Container  │        │
│  │  (Vercel)   │  │ Orchestrator│  │ Orchestrator│        │
│  └─────────────┘  │(Kubernetes) │  │(Kubernetes) │        │
│                    └─────────────┘  └─────────────┘        │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  MongoDB    │  │   Redis     │  │  RabbitMQ   │        │
│  │   Atlas     │  │   Cloud     │  │   Cloud     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

## Monitoring & Observability

### Logging Strategy
- **Structured Logging**: JSON format for all services
- **Log Levels**: Error, Warn, Info, Debug
- **Correlation IDs**: Request tracing across services
- **Centralized Logging**: ELK Stack or similar

### Metrics & Monitoring
- **Application Metrics**: Response times, error rates
- **Infrastructure Metrics**: CPU, memory, disk usage
- **Business Metrics**: User registrations, message counts
- **Real-time Dashboards**: Grafana or similar

### Health Checks
```javascript
// Health check endpoints for each service
GET /health
{
  "status": "healthy",
  "timestamp": "2023-09-01T12:00:00Z",
  "services": {
    "database": "connected",
    "redis": "connected",
    "rabbitmq": "connected"
  }
}
```

## Disaster Recovery

### Backup Strategy
- **Database Backups**: Daily MongoDB snapshots
- **Redis Persistence**: RDB + AOF for data durability
- **File Storage**: Cloudinary automatic backups
- **Configuration Backups**: Environment variables and configs

### Recovery Procedures
1. **Service Failure**: Auto-restart with health checks
2. **Database Failure**: Restore from latest backup
3. **Complete System Failure**: Multi-region deployment
4. **Data Corruption**: Point-in-time recovery

## Future Architecture Considerations

### Planned Enhancements
- **API Gateway**: Centralized routing and rate limiting
- **Service Mesh**: Inter-service communication management
- **Event Sourcing**: Audit trail and state reconstruction
- **CQRS**: Separate read/write operations
- **GraphQL**: Unified API layer

### Scalability Roadmap
- **Microservice Decomposition**: Further service splitting
- **Database Sharding**: Horizontal data partitioning
- **Caching Layers**: Multi-level caching strategy
- **Message Streaming**: Apache Kafka for high throughput
- **Global Distribution**: Multi-region deployment
