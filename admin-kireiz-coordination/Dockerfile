# syntax=docker/dockerfile:1

############################################
# Stage 1 - Install Dependencies
############################################
FROM node:20-alpine AS deps

WORKDIR /app

# Install compatibility libraries if needed
RUN apk add --no-cache libc6-compat

COPY package*.json ./

RUN npm ci

############################################
# Stage 2 - Build Application
############################################
FROM node:20-alpine AS builder

WORKDIR /app

ENV NEXT_TELEMETRY_DISABLED=1

COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build

############################################
# Stage 3 - Production Image
############################################
FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV PORT=7002
ENV HOSTNAME=0.0.0.0

# Create non-root user
RUN addgroup -g 1001 -S nodejs \
    && adduser -S nextjs -u 1001

# Copy standalone build
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./

# Copy static assets
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

USER nextjs

EXPOSE 7002

CMD ["node", "server.js"]
