---
title: "Deploying Ruby on Rails with Kamal v2 and Dokku: A Detailed Guide"
date: 2025-08-01T11:00:00-05:00
tags: ["ruby-on-rails", "kamal", "dokku", "deployment", "devops", "docker", "rails"]
draft: true
---

In the world of web development, deploying your application can be as complex as building it. For Ruby on Rails developers, the landscape of deployment tools is ever-evolving. While platforms like Heroku offer simplicity, they can become costly. Self-hosting provides control but often introduces complexity.

This guide explores a powerful, modern, and cost-effective middle ground: using **Kamal v2** for application deployment and **Dokku** for service management.

### The Power Couple: Kamal & Dokku

**Kamal** is a deployment tool created by the team behind Ruby on Rails. It brings the power of containers to your application without the steep learning curve of Kubernetes. Kamal performs zero-downtime deployments by orchestrating Docker containers on your own servers, using Traefik as a reverse proxy to handle requests and SSL certificates automatically.

**Dokku** is the original "Docker-powered mini-Heroku." It's a self-hosted Platform as a Service (PaaS) that excels at managing the lifecycle of services like databases (PostgreSQL, MySQL), caches (Redis), and more. It simplifies creating, linking, and backing up these essential components.

**Why use them together?**

By combining them, we get the best of both worlds:
-   **Kamal** handles the stateless part of our application—the Rails code itself—with modern, zero-downtime, container-based deploys.
-   **Dokku** handles the stateful services—our database and cache—providing a simple, Heroku-like CLI to manage them.

This setup gives you the power and control of self-hosting with a developer experience that is hard to beat.

### Prerequisites

Before we begin, make sure you have the following:

1.  **A Ruby on Rails Application:** A working app ready for production.
2.  **A Server:** A VPS from a provider like DigitalOcean, Hetzner, or Vultr running a fresh installation of Ubuntu 22.04.
3.  **A Domain Name:** A domain (e.g., `yourapp.com`) with an A record pointing to your server's IP address.
4.  **Local Environment:**
    -   Docker and Docker Compose installed.
    -   Ruby installed.
    -   SSH access to your server with a key.

### Part 1: Setting Up the Dokku Server

First, we'll configure our server with Dokku and set up the PostgreSQL and Redis services our Rails app will need.

#### 1.1 Install Dokku

SSH into your server and run the official bootstrap script. This will install Docker and all necessary Dokku components.

```/dev/null/install.sh#L1-2
wget https://dokku.com/install/v0.35.0/bootstrap.sh
sudo DOKKU_TAG=v0.35.0 bash bootstrap.sh
```

Once the installation is complete, open your browser and navigate to your server's IP address. You'll see the Dokku web installer.

-   Confirm your server's public key is correct.
-   Set the **Hostname** to your domain (e.g., `yourapp.com`).
-   Check "Use virtualhost naming for apps" to get nice subdomains like `my-app.yourapp.com`.

Finish the setup. You can now manage Dokku from your local machine via SSH.

#### 1.2 Install Service Plugins

We need plugins for PostgreSQL and Redis.

```/dev/null/dokku-plugins.sh#L1-2
sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres
sudo dokku plugin:install https://github.com/dokku/dokku-redis.git redis
```

#### 1.3 Create and Expose Services

Now, create the database and cache instances. We'll call them `rails-db` and `rails-cache`.

```/dev/null/dokku-services.sh#L1-2
# On your server
dokku postgres:create rails-db
dokku redis:create rails-cache
```

The crucial step for integrating with Kamal is to **expose** these services' ports to the host machine. This allows containers managed outside of Dokku (like our Kamal-deployed app) to connect to them.

```/dev/null/dokku-expose.sh#L1-2
# On your server
dokku postgres:expose rails-db 5432
dokku redis:expose rails-cache 6379
```

Dokku will map the container's internal port (e.g., 5432 for Postgres) to a random, high-numbered port on the server's host network interface.

#### 1.4 Get Connection URLs

Let's retrieve the connection details. Run these commands on your server:

```/dev/null/dokku-info.sh#L1-2
dokku postgres:info rails-db
dokku redis:info rails-cache
```

The output will look something like this:

```/dev/null/postgres-info.log#L1-6
=====> Postgres service information
       ...
       Dsn: postgres://postgres:RANDOM_PASSWORD@dokku-postgres-rails-db:5432/rails_db
       Exposed ports: 5432/tcp -> 32770
```

And for Redis:

```/dev/null/redis-info.log#L1-6
=====> Redis service information
       ...
       Url: redis://dokku-redis-rails-cache:RANDOM_PASSWORD@dokku-redis-rails-cache:6379
       Exposed ports: 6379/tcp -> 32771
```

From this, we construct our external connection URLs. Replace the service hostname (e.g., `dokku-postgres-rails-db`) with your server's public IP or domain and use the **exposed port**.

-   **Database URL:** `postgres://postgres:RANDOM_PASSWORD@yourapp.com:32770/rails_db`
-   **Redis URL:** `redis://:RANDOM_PASSWORD@yourapp.com:32771`

Keep these URLs handy. We'll add them to Kamal's configuration next.

### Part 2: Preparing the Rails App

Now, let's configure our local Rails application for deployment with Kamal.

#### 2.1 Add the Kamal Gem

Add `kamal` to your `Gemfile`:

```/dev/null/Gemfile#L1
gem "kamal", "~> 2.0"
```

Run `bundle install`.

#### 2.2 Initialize Kamal

Run the `init` command to generate the necessary configuration files.

```/dev/null/bash#L1
bundle exec kamal init
```

This creates two important files:
-   `config/deploy.yml`: The main deployment configuration file.
-   `.env`: A file to store secrets like registry passwords and your `RAILS_MASTER_KEY`. This file should be added to `.gitignore`.

Fill in the `.env` file with your `RAILS_MASTER_KEY` and credentials for a container registry (like Docker Hub or GitHub Container Registry).

```/dev/null/.env#L1-4
RAILS_MASTER_KEY=your_master_key_from_config/master.key
REGISTRY_USERNAME=your_registry_user
REGISTRY_PASSWORD=your_registry_access_token
```

#### 2.3 Configure `deploy.yml`

This is where we tell Kamal how to deploy our app. Open `config/deploy.yml` and let's customize it.

```kanthi.github.io/content/posts/rails-kamal-deployment.md/config/deploy.yml#L1-40
# Name of your application. Used to uniquely identify containers.
service: my-rails-app

# Name of the container image.
image: your_registry_user/my-rails-app

# Deploy to these servers.
servers:
  web:
    hosts:
      - 192.0.2.1 # Replace with your server's IP
    user: root # Or your user with sudo access

# Credentials for the container registry.
registry:
  username:
    - REGISTRY_USERNAME
  password:
    - REGISTRY_PASSWORD

# Environment variables.
env:
  # Send encrypted secrets to the server.
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL # We will set this via `kamal env push`
    - REDIS_URL    # We will set this via `kamal env push`
  # Clear-text variables can be added here.
  clear:
    RAILS_SERVE_STATIC_FILES: true

# Use Traefik as a reverse proxy to route traffic to the container.
traefik:
  options:
    publish:
      - "443:443"
      - "80:80"
  args:
    # Automatic HTTPS certificates from Let's Encrypt.
    entryPoints.web.address: ":80"
    entryPoints.websecure.address: ":443"
    entryPoints.web.http.redirections.entryPoint.to: websecure
    entryPoints.web.http.redirections.entryPoint.scheme: https
    certificatesResolvers.letsencrypt.acme.email: "your-email@example.com"
    certificatesResolvers.letsencrypt.acme.storage: "/letsencrypt/acme.json"
  hosts:
    - "my-rails-app.yourapp.com" # The domain for your app
  http:
    # This is required for Action Cable.
    middlewares:
      - "upgrade-headers@file"

# Health check to ensure the container is ready before switching traffic.
healthcheck:
  path: /up # Rails' default health check path
  port: 3000

# Mount a volume for SQLite3 database or other persistent files.
# volumes:
#   - "/path/on/host:/path/in/container"

# Run accessories on the same server, like a background worker.
# accessories:
#   worker:
#     cmd: bundle exec sidekiq
#     ...
```

#### 2.4 Push Environment Secrets

Instead of hardcoding the database and Redis URLs in `deploy.yml`, we'll push them securely using Kamal's `env` commands. This updates the `.env` file on the server.

```/dev/null/bash#L1-3
# Replace with the real URLs you got from Dokku
bundle exec kamal env secret DATABASE_URL="postgres://postgres:RANDOM_PASSWORD@yourapp.com:32770/rails_db"
bundle exec kamal env secret REDIS_URL="redis://:RANDOM_PASSWORD@yourapp.com:32771"
bundle exec kamal env push
```

This command encrypts your secrets and transfers them to the server, where they will be available to your container at runtime.

### Part 3: The Production Dockerfile

Kamal needs a `Dockerfile` to build your application image. Here is a robust, multi-stage `Dockerfile` optimized for production Rails applications.

```/dev/null/Dockerfile#L1-62
# syntax=docker/dockerfile:1

# 1. Base Stage
# ----------------
# Use the official Ruby image.
# You can specify a more specific version, e.g., ruby:3.3.0
FROM ruby:3.3-slim as base

# Set environment variables
ENV RAILS_ENV=production \
    BUNDLE_WITHOUT="development test" \
    BUNDLE_JOBS=4

# Install base dependencies
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends build-essential libvips git curl

# Set the working directory
WORKDIR /rails

# 2. Build Stage
# --------------
# Install gems and JS packages, precompile assets.
FROM base as build

# Install Node.js and Yarn (if needed for asset pipeline)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g yarn

# Copy Gemfile and package.json
COPY Gemfile Gemfile.lock package.json yarn.lock ./

# Install gems
RUN bundle install

# Install JS dependencies
RUN yarn install

# Copy the rest of the application code
COPY . .

# Precompile assets
RUN bundle exec rails assets:precompile

# 3. Production Stage
# -------------------
# Final image with only what's needed to run.
FROM base as production

# Create a non-root user to run the application
RUN useradd --create-home --shell /bin/bash app
USER app
WORKDIR /home/app/rails

# Copy installed gems and compiled assets from the build stage
COPY --from=build /usr/local/bundle/ /usr/local/bundle/
COPY --from=build /rails/public/assets/ /home/app/rails/public/assets/

# Copy application code
COPY --chown=app:app . .

# Expose port 3000
EXPOSE 3000

# Start the Rails server
CMD ["bundle", "exec", "rails", "server"]
```

### Part 4: Deploy!

With all the configuration in place, deploying is a two-step process.

#### 4.1 First-Time Setup

The `kamal setup` command prepares your server by installing Docker, logging into your registry, and starting the Traefik reverse proxy. You only need to run this once.

```/dev/null/bash#L1
bundle exec kamal setup
```

#### 4.2 Deploy the Application

Now, run the deploy command.

```/dev/null/bash#L1
bundle exec kamal deploy
```

Kamal will:
1.  Build your Docker image locally using the `Dockerfile`.
2.  Push the image to your container registry.
3.  Connect to your server and pull the new image.
4.  Run a temporary container to execute database migrations (`bin/rails db:migrate`).
5.  Perform a zero-downtime deploy by starting the new container, running a health check, and then stopping the old one. Traefik handles the traffic switch automatically.

Your Rails application is now live, served over HTTPS at the domain you configured!

### Conclusion

You now have a professional-grade deployment pipeline. Kamal provides a fantastic developer experience for deploying your Rails application, while Dokku acts as a reliable and easy-to-use manager for your database and cache.

This combination gives you the power of a custom cloud environment with the simplicity of a PaaS, all while keeping costs low. You can extend this setup by adding background workers as Kamal accessories, setting up Dokku volume backups, and integrating monitoring to create a truly robust production environment. Happy deploying!
