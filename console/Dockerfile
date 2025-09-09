# console/Dockerfile
FROM node:18.15.0-alpine as builder

# Create PNPM home dir
RUN mkdir -p ~/.pnpm
ENV PNPM_HOME=/root/.pnpm
ENV PATH=/root/.pnpm/bin:$PATH
ARG ENVIRONMENT=production
ARG API_HOST
# Make API_HOST available as environment variable for the build
ENV API_HOST=${API_HOST}

# Install global deps
RUN npm install -g ember-cli pnpm
RUN apk add --no-cache git openssh-client && \
  mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

# Set up workspace at root level
WORKDIR /workspace

# Copy workspace configuration from root
COPY pnpm-workspace.yaml pnpm-lock.yaml package.json ./

# Copy project directories
COPY packages ./packages
COPY console ./console

# Create the environments directory FIRST
RUN mkdir -p ./console/environments

# THEN copy .env file to where ember-cli-dotenv expects it
COPY .env ./console/environments/.env.${ENVIRONMENT}

# Install dependencies
RUN pnpm install --frozen-lockfile --shamefully-hoist

# Build from console directory
WORKDIR /workspace/console
RUN pnpm build --environment $ENVIRONMENT

# ---- Serve Stage ----
FROM nginx:alpine
COPY --from=builder /workspace/console/dist /usr/share/nginx/html
EXPOSE 4200
COPY console/nginx.conf /etc/nginx/conf.d/default.conf
CMD ["nginx", "-g", "daemon off;"]
