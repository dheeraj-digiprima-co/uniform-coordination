# syntax=docker/dockerfile:1

############################################
# Stage 1 - Install Dependencies
############################################
FROM node:20-alpine AS deps

WORKDIR /app

RUN apk add --no-cache libc6-compat

COPY package*.json ./

RUN npm install --legacy-peer-deps

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
ENV PORT=7003
ENV HOSTNAME=0.0.0.0

# Create non-root user
RUN addgroup -S nodejs -g 1001 && \
    adduser -S nextjs -u 1001 -G nodejs

# Copy required files
COPY --from=builder --chown=nextjs:nodejs /app/package*.json ./
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

USER nextjs

EXPOSE 7003

CMD ["npm", "start"]