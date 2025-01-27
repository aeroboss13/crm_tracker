# ----------------------- base -----------------
FROM node:18-bullseye AS base

USER root

# Update image and install necessary tools
RUN apt-get update && apt-get upgrade -y && apt-get install -y \
    g++ \
    make \
    python3 \
    curl

# Install libssl1.1
RUN curl -O http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.23_amd64.deb && \
    dpkg -i libssl1.1_1.1.1f-1ubuntu2.23_amd64.deb && \
    rm libssl1.1_1.1.1f-1ubuntu2.23_amd64.deb && \
    apt-get install -f -y

COPY . /app
WORKDIR /app

# Add NX
RUN yarn global add nx@latest

# ----------------------- dependencies -----------------
FROM base AS deps

# Install development dependencies
RUN npm install --only=development

# ------------------------ builder ---------------------
FROM base AS builder

WORKDIR /app

# Argument to select the env file
ARG ENV_FILE=.env
# Copy the env file
COPY ${ENV_FILE} .env

COPY --from=deps /app/node_modules ./node_modules

# Build the application
RUN yarn generate2
RUN yarn build:be

# ------------------------ runner ---------------------
FROM node:18-bullseye AS runner

WORKDIR /app

# Copy necessary files from builder
COPY --from=builder /app/dist/apps/backend ./
COPY ./packages/database/src/prisma/schema.prisma .
COPY ./yarn.lock .

# Copy environment file
COPY .env .env

# Install only production dependencies
RUN yarn install --production

# Ensure Prisma runtime dependencies are included
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma

# Expose application port
ENV PORT=3000
EXPOSE 3000

# Start the application
CMD ["node", "main.js"]
