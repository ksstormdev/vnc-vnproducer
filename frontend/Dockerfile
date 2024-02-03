FROM node:lts-alpine as base

FROM base as deps
WORKDIR /app
COPY package*.json ./
RUN npm install

FROM deps AS builder
WORKDIR /app
COPY . .
RUN npm run build
FROM deps AS prod-deps
WORKDIR /app
RUN npm ci --omit=dev

FROM base as runner
ENV NODE_ENV production
RUN apk add --no-cache libc6-compat bash nano curl wget iputils mysql-client redis && \
    addgroup --system --gid 1001 remix && \
    adduser --system --uid 1001 remix
USER remix
WORKDIR /app
COPY --from=prod-deps --chown=remix:remix /app/package*.json ./
COPY --from=prod-deps --chown=remix:remix /app/node_modules ./node_modules
COPY --from=builder --chown=remix:remix /app/build ./build
COPY --from=builder --chown=remix:remix /app/public ./public

ENTRYPOINT [ "node", "node_modules/.bin/remix-serve", "build/index.js"]