# Production Deployment Guide

This document outlines the requirements and steps for deploying the Visualizer application in a production environment.

## 1. Server Requirements
- **Ruby**: 4.0.1
- **PostgreSQL**: 17 or 18
- **Node.js**: Required for asset compilation (Tailwind CSS)

## 2. Environment Variables
Ensure the following variables are set in your production environment:
- `SECRET_KEY_BASE`: Secret key for Rails session encryption.
- `DATABASE_URL`: Connection string for PostgreSQL (e.g., `postgres://user:pass@host/dbname`).
- `RAILS_HOST`: The public domain or IP of your server (e.g., `visualizer.example.com`).
- `WEBAUTHN_ORIGIN`: The base URL including protocol (e.g., `https://visualizer.example.com`).
- `AWS_ACCESS_KEY`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`: Required if switching back to S3 storage.

## 3. Storage Configuration
The current configuration uses **local disk storage**.
- Ensure the `storage/` directory in the app root is writable by the web server user.
- **Persistence**: If using Docker or a transient server, ensure the `storage/` directory is mounted to a persistent volume.
- **Note**: For scalability, it is recommended to switch back to S3 by setting `config.active_storage.service = :amazon` in `config/environments/production.rb`.

## 4. Database Setup
1. Create the database:
   ```bash
   RAILS_ENV=production bin/rails db:prepare
   ```

## 5. Asset Compilation
Compile assets for production:
```bash
RAILS_ENV=production bin/rails assets:precompile
```

## 6. Web Server & Networking
- **SSL/HTTPS**: Production deployments should always use HTTPS. If you enable a load balancer or reverse proxy (like Nginx), update `config/environments/production.rb` to set `config.force_ssl = true`.
- **Binding**: The server should bind to `0.0.0.0` or the specific IP provided by your host.
- **Puma**: The application uses Puma by default. You can start it with:
  ```bash
  RAILS_ENV=production bundle exec puma -C config/puma.rb
  ```

---

## 7. 🚀 Local Performance Mode (Spare Laptop Setup)

If you are deploying on a spare laptop on your local network and want the **best performance** (equivalent to production), follow these steps to avoid the overhead of development mode:

### 1. Set Environment Variables
The production environment expects specific variables. For a local setup, you can set them in your shell or a `.env` file:
```bash
export RAILS_ENV=production
export RAILS_SERVE_STATIC_FILES=true
export SECRET_KEY_BASE=$(bin/rails secret)
export POSTGRES_HOST=localhost
export POSTGRES_PORT=5432
export POSTGRES_PASSWORD=your_postgres_password # If applicable
```

### 2. Prepare the Database (Production)
Instead of creating 5 development databases, this will only create the ones needed for production (`visualizer_production`, `visualizer_queue`, etc.):
```bash
bin/rails db:prepare
```

### 3. Precompile Assets
Production mode doesn't compile CSS/JS on the fly, making it much faster:
```bash
bin/rails assets:precompile
```

### 4. Start the Server
Use the production command, binding to all interfaces for network access:
```bash
bundle exec puma -p 3000 -b 0.0.0.0
```

> [!TIP]
> Running in `RAILS_ENV=production` disables code reloading and verbose logging, which significantly reduces CPU and RAM usage on your spare laptop.

## 8. Decent Espresso Machine Integration
For production, the hardcoded URL in the Decent Espresso `visualizer.tcl` plugin should point to your public domain. Users should use the standard plugin or a custom one pointing to:
`https://[YOUR_DOMAIN]/api/shots/upload`
