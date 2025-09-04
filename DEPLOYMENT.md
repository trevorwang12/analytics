# Plausible Analytics Deployment Guide for Dokploy

This guide covers deploying Plausible Analytics using Dokploy with optimized configuration.

## Important Notes

- **Elixir Version**: Modified to support Elixir 1.17+ (from original 1.18+ requirement) for Dokploy compatibility
- **Build System**: Can use either Nixpacks (automatic) or Dockerfile (manual configuration)

## Prerequisites

- Dokploy instance running
- Domain name pointing to your server
- SMTP credentials for email functionality
- SSL certificate (Dokploy can handle Let's Encrypt automatically)

## Quick Start

### Option 1: Simple Deployment (Recommended for limited disk space)

If your server has limited disk space, use the pre-built images:

1. **Use the simple docker-compose:**
   ```bash
   # Use docker-compose.simple.yml instead of building from source
   ```

2. **Configure environment variables in Dokploy:**
   - `BASE_URL`: Your domain (e.g., https://analytics.yoursite.com) 
   - `SECRET_KEY_BASE`: Generate a 64-character secret key
   - SMTP settings for email functionality
   - Database URLs are pre-configured

3. **Deploy with Dokploy:**
   - Import this repository into Dokploy
   - Use `docker-compose.simple.yml` as your compose file
   - Configure your domain and SSL
   - Deploy!

### Option 2: Build from Source (Requires more disk space)

For customization or latest features:

1. **Clone and prepare the repository:**
   ```bash
   git clone https://github.com/trevorwang12/analytics.git plausible-analytics
   cd plausible-analytics
   ```

2. **Configure environment variables:**
   ```bash
   cp .env.example .env
   ```
   Edit `.env` with your specific values

3. **Deploy with Dokploy:**
   - Use the main `docker-compose.yml`
   - Ensure your server has at least 4GB free space
   - Configure your domain and SSL

## Configuration Details

### Environment Variables

Key variables you must configure:

```env
BASE_URL=https://your-domain.com
SECRET_KEY_BASE=your-64-char-secret-key
DISABLE_REGISTRATION=invite_only
SMTP_HOST_ADDR=your-smtp-server.com
SMTP_USER_NAME=your-username
SMTP_USER_PWD=your-password
MAILER_EMAIL=analytics@your-domain.com
```

### Database Setup

The stack includes:
- **PostgreSQL 16**: Main application database
- **ClickHouse**: Analytics events storage

Both databases are configured with:
- Health checks
- Persistent volumes
- Automatic initialization

### Security Considerations

1. **Registration Control**: Set `DISABLE_REGISTRATION=invite_only`
2. **Strong Secrets**: Use `mix phx.gen.secret` for SECRET_KEY_BASE
3. **HTTPS Only**: Configure your domain with SSL in Dokploy
4. **Email Verification**: Configure SMTP for user verification

## Deployment Process

1. **Database Migration**: Automatic on first startup
2. **Health Checks**: Built-in health checks for all services
3. **Persistent Storage**: Data persists across deployments
4. **Rolling Updates**: Zero-downtime deployments

## Monitoring

### Health Checks

All services include health checks:
- Plausible app: `GET /api/health`
- PostgreSQL: `pg_isready`
- ClickHouse: `GET /ping`

### Logs

Access logs through Dokploy dashboard:
- Application logs: Main Plausible service
- Database logs: PostgreSQL and ClickHouse services

## Scaling

For high-traffic sites, consider:

1. **Resource Limits**: Adjust in `dokploy.yml`
   ```yaml
   resources:
     limits:
       memory: 1Gi
       cpu: 1000m
   ```

2. **Database Optimization**: 
   - Increase PostgreSQL shared_buffers
   - Configure ClickHouse memory settings

3. **Load Balancing**: Use Dokploy's load balancer features

## Troubleshooting

### Common Issues

1. **Disk Space Error** (`No space left on device`):
   - Use `docker-compose.simple.yml` instead of building from source
   - Clean up Docker images: `docker system prune -a`
   - Increase server disk space or use a VPS with more storage

2. **Database Connection**: Check DATABASE_URL and CLICKHOUSE_DATABASE_URL

3. **Email Issues**: Verify SMTP credentials and settings

4. **Domain Access**: Ensure domain is correctly configured in Dokploy

5. **Memory Issues**: Increase resource limits if needed

6. **Build Fails with Elixir Version**: 
   - Ensure you're using the compatible version (1.17+)
   - Consider using the pre-built image approach

### Debug Commands

Access application logs:
```bash
# Through Dokploy dashboard or CLI
dokploy logs plausible-analytics
```

Check database connectivity:
```bash
# PostgreSQL
docker exec -it plausible-db psql -U postgres -d plausible_db

# ClickHouse  
docker exec -it plausible-events-db clickhouse-client
```

## Backup Strategy

### Automated Backups

Configure in Dokploy:
1. **Database Backups**: Schedule PostgreSQL dumps
2. **Events Backup**: ClickHouse data export
3. **Volume Snapshots**: Persistent volume backups

### Manual Backup

```bash
# PostgreSQL backup
docker exec plausible-db pg_dump -U postgres plausible_db > backup.sql

# ClickHouse backup (requires configuration)
docker exec plausible-events-db clickhouse-backup create
```

## Updates

To update Plausible:

1. Pull latest code
2. Build new Docker image
3. Deploy through Dokploy (rolling update)
4. Database migrations run automatically

## Support

- Plausible Documentation: https://plausible.io/docs
- Dokploy Documentation: https://dokploy.com/docs
- Community Support: GitHub Issues

---

**Important**: Always test deployments in a staging environment first!