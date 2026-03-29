# Yahoo Chat (MERN + Socket.IO + Kubernetes)

A real-time chat application with:
- React frontend (Chakra UI)
- Node.js + Express backend
- MongoDB database
- Socket.IO for instant messaging and typing indicators
- Kubernetes manifests for cluster deployment

## Repository Structure

~~~text
yahoo-chat-prodv0.1/
├── backend/
│   ├── config/
│   │   ├── db.js
│   │   └── generateToken.js
│   ├── controllers/
│   │   ├── chatControllers.js
│   │   ├── messageControllers.js
│   │   └── userControllers.js
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   └── errorMiddleware.js
│   ├── models/
│   │   ├── chatModel.js
│   │   ├── messageModel.js
│   │   └── userModel.js
│   ├── routes/
│   │   ├── chatRoutes.js
│   │   ├── messageRoutes.js
│   │   └── userRoutes.js
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── config/
│   │   ├── Context/
│   │   ├── data/
│   │   └── Pages/
│   ├── Dockerfile
│   ├── package.json
│   └── README.md
├── k8s/
│   ├── namespace.yml
│   ├── mongodb-pv.yml
│   ├── mongodb-pvc.yml
│   ├── mongodb-deployment.yml
│   ├── mongodb-service.yml
│   ├── backend-deployment.yml
│   ├── backend-service.yml
│   ├── frontend-deployment.yml
│   ├── frontend-service.yml
│   └── ingress.yml
├── Dockerfile.frontend
├── package.json
└── README.md
~~~

## Features

- User registration and login with JWT authentication
- One-to-one chat
- Group chat creation and management
- Real-time message delivery using Socket.IO
- Typing indicator
- User search by name/email

## Bugs and Errors Fixed

The following fixes are present in the current main branch:

1. JWT/environment loading issue fixed
- Backend now explicitly loads environment variables from backend .env using:
  - dotenv.config({ path: __dirname + "/.env" })
- This avoids failures caused by running the server from a different working directory.

2. MongoDB integration compatibility fix
- Deprecated mongoose connect options were removed:
  - useNewUrlParser
  - useUnifiedTopology
- This resolves warning/error behavior with newer mongoose versions.

3. Message population compatibility fix
- Removed legacy execPopulate calls in message controller and switched to modern populate chaining.
- This fixes runtime issues with newer mongoose APIs while sending messages.

4. Group chat creation validation fix
- Group chat validation was corrected from requiring at least 2 other users to at least 1 other user.
- This matches expected UX for creating a group with creator + one participant.

5. Kubernetes deployment routing fix
- Backend production static-serving block was disabled to avoid route conflicts when frontend is served through NGINX service + ingress in Kubernetes.

6. Build/runtime dependency stability improvements
- Frontend uses:
  - --openssl-legacy-provider (react scripts)
  - --legacy-peer-deps for installation/build flows
- These changes reduce install/build failures in modern Node environments.

## Tech Stack

- Frontend: React 17, Chakra UI, Axios, Socket.IO Client
- Backend: Node.js, Express, Socket.IO, JWT
- Database: MongoDB
- Container: Docker
- Orchestration: Kubernetes + Ingress (NGINX)

## API Overview

Base API prefix: /api

- User
  - POST /api/user -> register
  - POST /api/user/login -> login
  - GET /api/user?search=<query> -> search users (auth required)

- Chat
  - POST /api/chat -> create/fetch one-to-one chat
  - GET /api/chat -> fetch chats
  - POST /api/chat/group -> create group
  - PUT /api/chat/rename -> rename group
  - PUT /api/chat/groupadd -> add member
  - PUT /api/chat/groupremove -> remove member

- Message
  - GET /api/message/:chatId -> get messages for a chat
  - POST /api/message -> send message

## Local Development Run

Prerequisites:
- Node.js 20+
- MongoDB running locally or remotely

1. Install dependencies

~~~bash
npm install
npm install --prefix frontend --legacy-peer-deps
~~~

2. Create backend environment file at backend/.env

~~~env
PORT=5000
MONGO_URI=mongodb://root:root@localhost:27017/yahoo-chat?authSource=admin
JWT_SECRET=replace_with_strong_secret
NODE_ENV=development
~~~

3. Start backend

~~~bash
npm run server
~~~

4. Start frontend (in another terminal)

~~~bash
npm start --prefix frontend
~~~

5. Open app
- Frontend: http://localhost:3000
- Backend health route (dev): http://localhost:5000/

## Docker Build and Run (Optional)

Backend image:

~~~bash
docker build -t yahoo-chat-backend:local ./backend
docker run -p 5000:5000 --env-file ./backend/.env yahoo-chat-backend:local
~~~

Frontend image:

~~~bash
docker build -t yahoo-chat-frontend:local -f frontend/Dockerfile .
docker run -p 8080:80 yahoo-chat-frontend:local
~~~

## Run on Kubernetes

Prerequisites:
- Kubernetes cluster (minikube/kind/cloud)
- kubectl configured
- NGINX Ingress Controller installed

### 1) Apply all manifests

~~~bash
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/mongodb-pv.yml
kubectl apply -f k8s/mongodb-pvc.yml
kubectl apply -f k8s/mongodb-deployment.yml
kubectl apply -f k8s/mongodb-service.yml
kubectl apply -f k8s/backend-deployment.yml
kubectl apply -f k8s/backend-service.yml
kubectl apply -f k8s/frontend-deployment.yml
kubectl apply -f k8s/frontend-service.yml
kubectl apply -f k8s/ingress.yml
~~~

### 2) Verify resources

~~~bash
kubectl get all -n yahoo-chat
kubectl get ingress -n yahoo-chat
kubectl get pvc,pv -n yahoo-chat
~~~

### 3) Access the application

Ingress is configured with host localhost and routes:
- / -> frontend-service
- /api -> backend-service

For local clusters:
- If using minikube, run:

~~~bash
minikube addons enable ingress
minikube tunnel
~~~

Then open:
- http://localhost/

If your ingress controller exposes a different IP/host, update k8s/ingress.yml accordingly.

## How to Use the App

1. Open the app home page.
2. Sign up with name, email, and password.
3. Log in and search users from the sidebar.
4. Click a user to start one-to-one chat.
5. Use New Group to create a group chat.
6. Type and send messages in real time.
7. Observe typing indicators while users are typing.

## Common Troubleshooting

1. Backend cannot connect to MongoDB
- Check MONGO_URI in backend/.env.
- If using authenticated MongoDB root user, keep authSource=admin.

2. 401 Unauthorized on protected endpoints
- Ensure token is sent in Authorization header:
  - Authorization: Bearer <token>

3. Frontend build fails due to peer/OpenSSL issues
- Use npm install --legacy-peer-deps.
- Keep frontend scripts with --openssl-legacy-provider.

4. Kubernetes app not reachable
- Confirm ingress controller is installed and running.
- Confirm ingressClassName: nginx matches your controller.
- Check frontend-service type (LoadBalancer) behavior on your cluster.

## Notes

- Current Kubernetes manifests reference prebuilt public images:
  - arnaba075/chatapp-backend:v5
  - arnaba075/chatapp-frontend:latest
- If you build your own images, update image names in deployment manifests before applying.
