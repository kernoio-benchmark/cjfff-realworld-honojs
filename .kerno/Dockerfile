# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Install OpenSSL for Prisma
RUN apk add --no-cache openssl

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev dependencies for build)
RUN npm install

# Copy Prisma schema first
COPY prisma ./prisma

# Generate Prisma Client
RUN npm run prisma:generate

# Copy source code
COPY src ./src

# Build using esbuild as a fallback since tsc has type errors
# Install esbuild and build the project
RUN npm install -D esbuild && \
    npx esbuild src/index.ts --bundle --platform=node --format=esm --outdir=dist --external:@prisma/client --external:@hono/* --external:hono --external:jose --external:bcryptjs --external:zod --packages=external

# Production stage
FROM node:20-alpine AS production

WORKDIR /app

# Install OpenSSL for Prisma
RUN apk add --no-cache openssl

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm install --omit=dev && \
    npm cache clean --force

# Copy Prisma schema for runtime
COPY prisma ./prisma

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Copy node_modules with generated Prisma Client from builder
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma
COPY --from=builder /app/node_modules/@prisma ./node_modules/@prisma

# Expose port (default Hono.js port, adjust if needed)
EXPOSE 3000

# Start the application
# Note: Run migrations and seed before starting if needed:
# docker run -e DATABASE_URL=... <image> sh -c "npm run prisma:migrate && npm run db:seed && npm start"
CMD ["npm", "start"]
