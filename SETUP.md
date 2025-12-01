# Setup Guide for Portainer Deployment

This guide will help you deploy the AppFlowy Task Management Server using Portainer.

## Prerequisites

1. Portainer installed and running
2. Git repository access
3. Docker and Docker Compose installed on the host

## Step-by-Step Setup

### 1. Prepare Your Repository

Make sure all files are committed and pushed to your GitHub repository:
- `docker-compose.yml`
- `backend/Dockerfile`
- `frontend/Dockerfile`
- `config/` directory
- `.env.example`

### 2. Create Environment File

In Portainer, you'll need to set environment variables. Create a `.env` file locally first to test, then add these to Portainer:

```bash
POSTGRES_DB=appflowy
POSTGRES_USER=appflowy
POSTGRES_PASSWORD=your_secure_password_here
APPFLOWY_SERVER_URL=http://your-domain-or-ip:6633
APPFLOWY_WS_URL=ws://your-domain-or-ip:6633/ws
JWT_SECRET=generate_with_openssl_rand_base64_32
RUST_LOG=info
STORAGE_PATH=/app/storage
```

### 3. Deploy in Portainer

#### Option A: Using Repository (Recommended)

1. Log into Portainer
2. Navigate to **Stacks** in the left sidebar
3. Click **Add Stack**
4. Name your stack: `appflowy-task-manager`
5. Select **Repository** method
6. Configure:
   - **Repository URL**: Your GitHub repository URL
   - **Repository Reference**: `main` or your branch name
   - **Compose path**: `docker-compose.yml`
   - **Auto-update**: Enable if you want automatic updates
7. Scroll down to **Environment variables**
8. Add all variables from your `.env` file
9. Click **Deploy the stack**

#### Option B: Using Web Editor

1. Log into Portainer
2. Navigate to **Stacks**
3. Click **Add Stack**
4. Name your stack: `appflowy-task-manager`
5. Select **Web editor** method
6. Copy the contents of `docker-compose.yml` into the editor
7. Add environment variables in the **Environment variables** section
8. Click **Deploy the stack**

### 4. Verify Deployment

1. Check stack status in Portainer - all services should show as "Running"
2. Check logs for any errors:
   - Click on your stack
   - Click on individual services to view logs
3. Test the health endpoint: `http://your-server:6633/health`
4. Access the application: `http://your-server:6633`

### 5. Access Storage Directory

User files are stored in the `storage/` directory. If you need to access it:

1. The directory is mounted at `./storage` relative to where you cloned the repo
2. Or access via Docker volume: `appflowy_storage`
3. In Portainer: **Volumes** > `appflowy_storage` > **Inspect**

## Troubleshooting

### Services Won't Start

1. **Check logs**: View logs for each service in Portainer
2. **Check environment variables**: Ensure all required variables are set
3. **Check port availability**: Ensure port 6633 is not in use
4. **Check database**: Verify PostgreSQL is healthy before backend starts

### Build Failures

If the Docker build fails:
1. Check if AppFlowy repository structure has changed
2. Verify Dockerfiles are correct
3. Check build logs in Portainer

### Connection Issues

1. **Backend can't connect to database**:
   - Verify `DATABASE_URL` environment variable
   - Check PostgreSQL service is running
   - Verify network connectivity

2. **Frontend can't connect to backend**:
   - Verify `APPFLOWY_SERVER_URL` and `APPFLOWY_WS_URL`
   - Check Nginx configuration
   - Verify all services are on the same network

### Update Application

1. In Portainer, go to your stack
2. Click **Editor**
3. Pull latest changes or update configuration
4. Click **Update the stack**

## Security Recommendations

1. **Change default passwords** immediately
2. **Generate strong JWT_SECRET**: `openssl rand -base64 32`
3. **Use HTTPS** in production (configure SSL in Nginx)
4. **Restrict access** to Portainer and the application
5. **Regular backups** of database and storage

## Backup and Restore

### Backup Database

```bash
docker exec appflowy-postgres pg_dump -U appflowy appflowy > backup.sql
```

### Restore Database

```bash
docker exec -i appflowy-postgres psql -U appflowy appflowy < backup.sql
```

### Backup Storage

Simply copy the `storage/` directory to your backup location.

## Next Steps

- Configure custom domain (if needed)
- Set up SSL/TLS certificates
- Configure firewall rules
- Set up automated backups
- Plan for Dropbox/Google Docs integration

