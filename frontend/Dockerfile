# Build stage
FROM node:20-alpine AS build
WORKDIR /app

# Copy frontend package manifests
COPY frontend/package.json frontend/package-lock.json ./

# Install dependencies
RUN npm ci --legacy-peer-deps

# Copy frontend source
COPY frontend ./

# Build production static assets
RUN npm run build

# Production stage
FROM nginx:alpine AS production
# Remove default nginx website
RUN rm -rf /usr/share/nginx/html/*

# Copy build output from the previous stage
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
