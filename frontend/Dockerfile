# Build stage
FROM node:18.20.8-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:18.20.8-alpine

# Cài serve để chạy static files
RUN npm install -g serve

WORKDIR /app

# Copy build từ stage trước
COPY --from=builder /app/build ./build

EXPOSE 3000

# Chạy app production trên port 3000
CMD ["serve", "-s", "build", "-l", "3000"]
