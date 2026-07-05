# Stage 1: Build dependency graph and compile frontend
FROM node:20.11-alpine AS builder
WORKDIR /app

# Copy dependency configs
COPY package.json package-lock.json* ./
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi

# Copy application code
COPY . .

# Set build arguments for inlining env vars
ARG NEXT_PUBLIC_API_URL=http://localhost:8000
ARG NEXT_PUBLIC_WS_URL=ws://localhost:8000
ARG NEXT_PUBLIC_SITE_URL=http://localhost:3000
ARG NEXT_PUBLIC_S3_BUCKET_DOMAIN=
ARG NEXT_PUBLIC_CLOUDFRONT_DOMAIN=

ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL \
    NEXT_PUBLIC_WS_URL=$NEXT_PUBLIC_WS_URL \
    NEXT_PUBLIC_SITE_URL=$NEXT_PUBLIC_SITE_URL \
    NEXT_PUBLIC_S3_BUCKET_DOMAIN=$NEXT_PUBLIC_S3_BUCKET_DOMAIN \
    NEXT_PUBLIC_CLOUDFRONT_DOMAIN=$NEXT_PUBLIC_CLOUDFRONT_DOMAIN

# Build application with output standalone
RUN npm run build

# Stage 2: Minimal runtime environment
FROM node:20.11-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

# Don't run as root for security
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Copy only the compiled standalone app and assets
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

# Run the standalone Node app (much faster, no devDependencies or build tools)
CMD ["node", "server.js"]
