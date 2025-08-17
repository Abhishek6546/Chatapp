# Chat App

A modern real-time chat application built with microservices architecture, featuring user authentication, real-time messaging, and email verification.

## 🏗️ Architecture

This application follows a microservices architecture with the following components:

### Backend Services
- **User Service** (Port: configurable) - Handles user authentication, registration, and profile management
- **Chat Service** (Port: configurable) - Manages real-time messaging with Socket.IO
- **Mail Service** (Port: configurable) - Handles email notifications and OTP verification

### Frontend
- **Next.js Application** - Modern React-based frontend with TypeScript and Tailwind CSS

### Infrastructure
- **MongoDB** - Primary database for user data and chat messages
- **Redis** - Caching and session management
- **RabbitMQ** - Message queue for inter-service communication
- **Cloudinary** - Image and file upload handling

## 🚀 Features

- **Real-time Messaging** - Instant messaging with Socket.IO
- **User Authentication** - JWT-based authentication with email verification
- **File Sharing** - Image and file upload support via Cloudinary
- **Responsive Design** - Modern UI with Tailwind CSS
- **Microservices** - Scalable architecture with independent services
- **Message Queue** - Reliable inter-service communication with RabbitMQ

## 🛠️ Tech Stack

### Frontend
- Next.js 15.3.4
- React 19
- TypeScript
- Tailwind CSS
- Socket.IO Client
- Axios
- React Hot Toast
- Lucide React (Icons)

### Backend
- Node.js with Express
- TypeScript
- Socket.IO
- MongoDB with Mongoose
- Redis
- RabbitMQ (amqplib)
- JWT Authentication
- Cloudinary
- Nodemailer
- Multer (File uploads)

## 📋 Prerequisites

Before running this application, make sure you have the following installed:

- Node.js (v18 or higher)
- MongoDB
- Redis
- RabbitMQ
- npm or yarn

## ⚙️ Environment Variables

Create `.env` files in each service directory with the following variables:

### Backend Services (.env)
```env
# Database
MONGO_URI=mongodb://localhost:27017/chatapp
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your_jwt_secret_key

# RabbitMQ
RABBITMQ_URL=amqp://localhost

# Cloudinary (for file uploads)
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret

# Email (for OTP)
EMAIL_USER=your_email@gmail.com
EMAIL_PASS=your_app_password

# Ports
PORT=3001  # User service
PORT=3002  # Chat service  
PORT=3003  # Mail service
```

### Frontend (.env.local)
```env
NEXT_PUBLIC_USER_API_URL=http://localhost:3001/api/v1
NEXT_PUBLIC_CHAT_API_URL=http://localhost:3002/api/v1
NEXT_PUBLIC_SOCKET_URL=http://localhost:3002
```

## 🚀 Installation & Setup

### 1. Clone the repository
```bash
git clone <repository-url>
cd chatapp
```

### 2. Install dependencies for all services

#### Frontend
```bash
cd frontend
npm install
```

#### Backend Services
```bash
# User Service
cd backend/user
npm install

# Chat Service
cd ../chat
npm install

# Mail Service
cd ../mail
npm install
```

### 3. Start Infrastructure Services

Make sure MongoDB, Redis, and RabbitMQ are running:

```bash
# Start MongoDB
mongod

# Start Redis
redis-server

# Start RabbitMQ
rabbitmq-server
```

### 4. Build and Start Backend Services

```bash
# User Service
cd backend/user
npm run build
npm run dev

# Chat Service (in new terminal)
cd backend/chat
npm run build
npm run dev

# Mail Service (in new terminal)
cd backend/mail
npm run build
npm run dev
```

### 5. Start Frontend
```bash
cd frontend
npm run dev
```

## 📱 Usage

1. **Access the Application**: Open http://localhost:3000 in your browser
2. **Register**: Create a new account with email verification
3. **Login**: Sign in with your credentials
4. **Start Chatting**: Join chat rooms and start messaging in real-time
5. **Share Files**: Upload and share images through the chat interface

## 🏃‍♂️ Development

### Available Scripts

#### Frontend
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

#### Backend Services
- `npm run dev` - Start development server with hot reload
- `npm run build` - Compile TypeScript
- `npm start` - Start production server

## 🔧 API Endpoints

### User Service (Port 3001)
- `POST /api/v1/register` - User registration
- `POST /api/v1/login` - User login
- `POST /api/v1/verify-otp` - Email verification
- `GET /api/v1/profile` - Get user profile
- `PUT /api/v1/profile` - Update user profile

### Chat Service (Port 3002)
- `GET /api/v1/chats` - Get user chats
- `POST /api/v1/chat` - Create new chat
- `GET /api/v1/messages/:chatId` - Get chat messages
- `POST /api/v1/message` - Send message
- Socket.IO events for real-time messaging

## 🐳 Docker Support

Docker configuration can be added for containerized deployment. Consider creating:
- `Dockerfile` for each service
- `docker-compose.yml` for orchestration
- Environment-specific configurations

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the ISC License.

## 🔍 Troubleshooting

### Common Issues

1. **Connection Refused Errors**: Ensure all infrastructure services (MongoDB, Redis, RabbitMQ) are running
2. **CORS Issues**: Check that frontend and backend URLs are correctly configured
3. **Socket Connection Issues**: Verify Socket.IO server is running and ports are correct
4. **File Upload Issues**: Ensure Cloudinary credentials are properly configured

### Logs

Check individual service logs for debugging:
```bash
# View logs for specific service
cd backend/[service-name]
npm run dev
```

## 📞 Support

For support and questions, please open an issue in the repository or contact the development team.
