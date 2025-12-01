# AppFlowy Task Management Server

A self-hosted AppFlowy server tailored for task management, deployed via Docker and Portainer.

## Features

- ✅ Full AppFlowy stack (Backend + Frontend)
- ✅ PostgreSQL database
- ✅ Task management with nesting support
- ✅ JWT-based authentication (self-contained, no external services)
- ✅ Local file storage (accessible outside Docker)
- ✅ Health check endpoints
- ✅ Ready for Portainer deployment

## Prerequisites

- Docker and Docker Compose installed
- Portainer (for deployment)
- Git (for repository cloning)

## Quick Start

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd "Task Manager"
```

### 2. Configure Environment Variables

```bash
cp .env.example .env
```

Edit `.env` and update the following:
- `POSTGRES_PASSWORD`: Set a strong password
- `JWT_SECRET`: Generate a secure secret (use `openssl rand -base64 32`)
- `APPFLOWY_SERVER_URL`: Update if accessing from a different host/domain

### 3. Create Storage Directory

```bash
mkdir -p storage
chmod 755 storage
```

### 4. Deploy with Docker Compose

```bash
docker-compose up -d
```

### 5. Access the Application

- Frontend: http://localhost:6633
- Health Check: http://localhost:6633/health
- Backend API: http://localhost:6633/api

## Portainer Setup

1. In Portainer, go to **Stacks**
2. Click **Add Stack**
3. Name your stack (e.g., `appflowy-task-manager`)
4. Choose **Repository** method
5. Enter your Git repository URL
6. Set the path to `docker-compose.yml`
7. Configure environment variables in Portainer or use `.env` file
8. Click **Deploy the stack**

### Portainer Environment Variables

You can set environment variables directly in Portainer:
- Go to your stack
- Click **Editor**
- Add environment variables in the `env` section

## Project Structure

```
.
├── backend/              # AppFlowy backend
│   └── Dockerfile
├── frontend/            # AppFlowy frontend
│   └── Dockerfile
├── config/              # Configuration files
│   ├── nginx/          # Nginx configuration
│   └── postgres/       # PostgreSQL initialization
├── storage/            # User files (mounted volume)
├── docker-compose.yml  # Main Docker Compose file
├── .env.example        # Environment variables template
└── README.md          # This file
```

## Configuration

### Port Configuration

The application runs on port **6633** by default. To change it:
1. Update `docker-compose.yml` ports mapping: `"NEW_PORT:6633"`
2. Update `APPFLOWY_SERVER_URL` and `APPFLOWY_WS_URL` in `.env`

### Database

PostgreSQL runs on the internal network. Connection details:
- Host: `postgres` (internal)
- Port: `5432`
- Database: Set in `POSTGRES_DB` env var
- User: Set in `POSTGRES_USER` env var
- Password: Set in `POSTGRES_PASSWORD` env var

### File Storage

User-uploaded files are stored in the `./storage` directory on the host machine, making them easily accessible outside Docker.

### Authentication

The system uses JWT-based authentication:
- No external authentication services required
- Self-contained and secure
- Tokens are signed with `JWT_SECRET`

**Important**: Change `JWT_SECRET` in production!

## Health Checks

- **Application**: `http://localhost:6633/health`
- **Backend**: `http://localhost:6633/api/health`
- **Database**: Automatically checked by Docker health checks

## Maintenance

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f postgres
```

### Backup Database

```bash
docker-compose exec postgres pg_dump -U appflowy appflowy > backup.sql
```

### Restore Database

```bash
docker-compose exec -T postgres psql -U appflowy appflowy < backup.sql
```

### Update Application

```bash
git pull
docker-compose build --no-cache
docker-compose up -d
```

### Stop Application

```bash
docker-compose down
```

### Stop and Remove Volumes (⚠️ Deletes Data)

```bash
docker-compose down -v
```

## Troubleshooting

### Port Already in Use

If port 6633 is already in use:
1. Change the port in `docker-compose.yml`
2. Update `APPFLOWY_SERVER_URL` and `APPFLOWY_WS_URL` in `.env`

### Database Connection Issues

1. Check PostgreSQL is healthy: `docker-compose ps`
2. Check logs: `docker-compose logs postgres`
3. Verify environment variables in `.env`

### Storage Permission Issues

```bash
sudo chown -R $USER:$USER storage/
chmod -R 755 storage/
```

## Security Notes

1. **Change default passwords** in `.env`
2. **Generate a strong JWT_SECRET** using `openssl rand -base64 32`
3. **Use HTTPS** in production (configure Nginx SSL)
4. **Restrict access** to the server if exposed to the internet
5. **Regular backups** of the database and storage directory

## Future Enhancements

- [ ] Dropbox integration
- [ ] Google Docs integration
- [ ] Advanced task management features
- [ ] Multi-user collaboration
- [ ] Email notifications

## License

This project uses AppFlowy, which is licensed under the AGPL-3.0 license.

## Support

For issues related to:
- **AppFlowy**: Check [AppFlowy Documentation](https://docs.appflowy.io)
- **This Setup**: Open an issue in this repository

