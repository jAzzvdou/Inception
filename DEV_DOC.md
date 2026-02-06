# Developer Documentation - Inception Project

This document provides technical instructions for developers on how to set up, build, and manage the Inception Docker infrastructure from scratch.

## Prerequisites

Before setting up the project, ensure you have:

1. **Linux Virtual Machine** (required by project specifications)
2. **Docker** (version 20.10 or later)
3. **Docker Compose v2** (included with Docker Desktop or install separately)
4. **Git** (for cloning the repository)
5. **Root or sudo access** (for system configuration)
6. **Basic knowledge of:**
   - Docker and containerization concepts
   - YAML syntax (for docker-compose.yml)
   - Shell scripting basics
   - Linux system administration

### Installing Docker and Docker Compose

If Docker is not installed:

```bash
# Update package index
sudo apt update

# Install prerequisites
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up the repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine and Docker Compose
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your user to the docker group
sudo usermod -aG docker $USER

# Apply group changes (logout/login or use newgrp)
newgrp docker

# Verify installation
docker --version
docker compose version
```

## Project Setup from Scratch

### 1. Repository Structure

The project must follow this structure:

```
inception/
├── Makefile
├── secrets/
│   ├── db_root_password.txt
│   ├── wp_db_password.txt
│   ├── wp_admin_password.txt
│   └── wp_user2_password.txt
└── srcs/
    ├── docker-compose.yml
    ├── .env
    └── requirements/
        ├── nginx/
        │   ├── Dockerfile
        │   └── conf/
        │       └── default.conf
        ├── wordpress/
        │   ├── Dockerfile
        │   ├── conf/
        │   │   └── www.conf
        │   └── tools/
        │       └── wp-cli_script.sh
        └── mariadb/
            ├── Dockerfile
            ├── conf/
            │   └── 50-server.cnf
            └── tools/
                └── mariadb_script.sh
```

### 2. Configuration Files

#### Creating the `.env` File

Create `inception/srcs/.env` with the following variables (replace `jazevedo` with your login):

```env
# Domain configuration
SERVER_NAME=jazevedo.42.fr

# Database configuration
DB_HOST=mariadb
DB_NAME=wordpress
DB_USER=wpuser

# WordPress configuration
WP_URL=https://jazevedo.42.fr
WP_TITLE=Inception

# Administrator user (username cannot contain "admin" or "Administrator")
WP_ADMIN_USER=super_jazevedo
WP_ADMIN_EMAIL=jazevedo@gmail.com

# Secondary user
WP_USER2=jazevedo
WP_USER2_EMAIL=jazevedo2@gmail.com
WP_USER2_ROLE=author
```

#### Creating Docker Secrets

Create the `inception/secrets/` directory and add password files:

```bash
mkdir -p inception/secrets

# Create secret files (replace with strong passwords)
echo "your_secure_root_password" > inception/secrets/db_root_password.txt
echo "your_secure_db_password" > inception/secrets/wp_db_password.txt
echo "your_secure_admin_password" > inception/secrets/wp_admin_password.txt
echo "your_secure_user2_password" > inception/secrets/wp_user2_password.txt

# Set proper permissions
chmod 600 inception/secrets/*.txt
```

**⚠️ Security Warning**: Never commit these files to Git. Ensure they are in `.gitignore`.

#### Configuring Domain Resolution

Add your domain to `/etc/hosts`:

```bash
sudo echo "127.0.0.1 jazevedo.42.fr" >> /etc/hosts
```

Replace `jazevedo` with your actual login.

### 3. Creating Data Directories

Create the data directories for volumes:

```bash
mkdir -p /home/jazevedo/data/wordpress
mkdir -p /home/jazevedo/data/mariadb

# Set appropriate permissions
sudo chown -R $USER:$USER /home/jazevedo/data
```

Replace `jazevedo` with your login.

## Building and Launching the Project

### Using the Makefile

The Makefile provides convenient commands:

```bash
cd inception

# Build and start all services
make
# or
make inception

# Stop all services
make clean

# Complete cleanup (removes containers, images, volumes, networks)
make fclean

# Rebuild everything from scratch
make re
```

### Using Docker Compose Directly

You can also use Docker Compose commands directly:

```bash
cd inception/srcs

# Build and start services
docker compose up -d --build

# View logs
docker compose logs -f

# Stop services
docker compose down

# Stop and remove volumes
docker compose down -v
```

### Build Process

When you run `make`, the following happens:

1. **Docker Compose reads `docker-compose.yml`**
2. **Builds images** from Dockerfiles in `requirements/` directories
3. **Creates network** (`inception`)
4. **Creates volumes** (wordpress_data, mariadb_data)
5. **Starts containers** in dependency order:
   - MariaDB starts first
   - WordPress waits for MariaDB
   - NGINX waits for both

### First Startup Sequence

1. **MariaDB container**:
   - Initializes database if `/var/lib/mysql/${MYSQL_DATABASE}` doesn't exist
   - Creates database and user from environment variables
   - Starts mysqld_safe

2. **WordPress container**:
   - Waits for MariaDB to be ready (mysqladmin ping)
   - Downloads WordPress core if not present
   - Creates `wp-config.php` with database credentials
   - Installs WordPress via WP-CLI
   - Creates administrator and secondary users
   - Starts php-fpm8.2

3. **NGINX container**:
   - Generates self-signed SSL certificate during build
   - Starts nginx with SSL on port 443

## Managing Containers and Volumes

### Container Management

**List running containers:**
```bash
docker compose -f srcs/docker-compose.yml ps
```

**View container logs:**
```bash
# All services
docker compose -f srcs/docker-compose.yml logs

# Specific service
docker compose -f srcs/docker-compose.yml logs nginx
docker compose -f srcs/docker-compose.yml logs wp-php
docker compose -f srcs/docker-compose.yml logs mariadb

# Follow logs in real-time
docker compose -f srcs/docker-compose.yml logs -f
```

**Execute commands in containers:**
```bash
# Access WordPress container shell
docker compose -f srcs/docker-compose.yml exec wp-php bash

# Access MariaDB container shell
docker compose -f srcs/docker-compose.yml exec mariadb bash

# Run WP-CLI commands
docker compose -f srcs/docker-compose.yml exec wp-php ./wp-cli.phar --info
```

**Restart a specific service:**
```bash
docker compose -f srcs/docker-compose.yml restart nginx
```

**Rebuild a specific service:**
```bash
docker compose -f srcs/docker-compose.yml up -d --build nginx
```

### Volume Management

**List volumes:**
```bash
docker volume ls
```

**Inspect volume details:**
```bash
docker volume inspect inception_wordpress_data
docker volume inspect inception_mariadb_data
```

**View volume mount points:**
The volumes are backed by bind mounts to:
- WordPress: `/home/jazevedo/data/wordpress`
- MariaDB: `/home/jazevedo/data/mariadb`

**Backup volumes:**
```bash
# Backup WordPress files
tar -czf wordpress_backup.tar.gz /home/jazevedo/data/wordpress

# Backup MariaDB (from inside container)
docker compose -f srcs/docker-compose.yml exec mariadb mysqldump -u root -p wordpress > backup.sql
```

**Remove volumes (⚠️ deletes data):**
```bash
docker compose -f srcs/docker-compose.yml down -v
```

### Network Management

**List networks:**
```bash
docker network ls
```

**Inspect network:**
```bash
docker network inspect inception_inception
```

**View connected containers:**
```bash
docker network inspect inception_inception | grep -A 10 Containers
```

## Data Persistence

### How Data Persists

Data persistence is achieved through Docker named volumes backed by bind mounts:

1. **WordPress Volume** (`wordpress_data`):
   - Mounted at `/var/www/html` in the WordPress container
   - Backed by `/home/jazevedo/data/wordpress` on the host
   - Contains: WordPress core files, themes, plugins, uploads, `wp-config.php`

2. **MariaDB Volume** (`mariadb_data`):
   - Mounted at `/var/lib/mysql` in the MariaDB container
   - Backed by `/home/jazevedo/data/mariadb` on the host
   - Contains: All database files, tables, and data

### Data Location on Host

All persistent data is stored in:
```
/home/jazevedo/data/
├── wordpress/    # WordPress files
└── mariadb/      # Database files
```

### Verifying Persistence

1. **Make a change** (create a post, upload media, etc.)
2. **Stop containers:**
   ```bash
   cd inception
   make clean
   ```
3. **Restart containers:**
   ```bash
   make
   ```
4. **Verify changes persist** - your data should still be there

### Data Backup Strategy

**WordPress files backup:**
```bash
tar -czf wordpress_backup_$(date +%Y%m%d).tar.gz /home/jazevedo/data/wordpress
```

**Database backup:**
```bash
docker compose -f srcs/docker-compose.yml exec mariadb \
  mysqldump -u root -p$(cat ../secrets/db_root_password.txt) \
  wordpress > wordpress_db_$(date +%Y%m%d).sql
```

**Restore database:**
```bash
docker compose -f srcs/docker-compose.yml exec -T mariadb \
  mysql -u root -p$(cat ../secrets/db_root_password.txt) \
  wordpress < wordpress_db_20240101.sql
```

## Development Workflow

### Making Configuration Changes

1. **Edit configuration files** (e.g., `nginx/conf/default.conf`)
2. **Rebuild the affected service:**
   ```bash
   docker compose -f srcs/docker-compose.yml up -d --build nginx
   ```

### Debugging

**Check container status:**
```bash
docker compose -f srcs/docker-compose.yml ps
```

**View detailed logs:**
```bash
docker compose -f srcs/docker-compose.yml logs --tail=100 -f
```

**Inspect container:**
```bash
docker inspect nginx
docker inspect wp-php
docker inspect mariadb
```

**Check environment variables:**
```bash
docker compose -f srcs/docker-compose.yml exec nginx env
```

**Test database connection:**
```bash
docker compose -f srcs/docker-compose.yml exec mariadb \
  mysql -u wpuser -p$(cat ../../secrets/wp_db_password.txt) wordpress
```

### Common Development Tasks

**Update WordPress:**
```bash
docker compose -f srcs/docker-compose.yml exec wp-php \
  ./wp-cli.phar core update --allow-root
```

**Install a plugin:**
```bash
docker compose -f srcs/docker-compose.yml exec wp-php \
  ./wp-cli.phar plugin install <plugin-name> --activate --allow-root
```

**Clear WordPress cache:**
```bash
docker compose -f srcs/docker-compose.yml exec wp-php \
  ./wp-cli.phar cache flush --allow-root
```

**Access MariaDB directly:**
```bash
docker compose -f srcs/docker-compose.yml exec mariadb \
  mysql -u root -p$(cat ../../secrets/db_root_password.txt)
```

## Troubleshooting

### Containers Won't Start

1. **Check Docker daemon:**
   ```bash
   docker ps
   ```

2. **Check for port conflicts:**
   ```bash
   sudo netstat -tlnp | grep 443
   ```

3. **Check disk space:**
   ```bash
   df -h
   ```

4. **View error logs:**
   ```bash
   docker compose -f srcs/docker-compose.yml logs
   ```

### WordPress Installation Page Appears

This means WordPress wasn't configured. Check:

1. **Database connection:**
   ```bash
   docker compose -f srcs/docker-compose.yml logs mariadb
   ```

2. **WordPress container logs:**
   ```bash
   docker compose -f srcs/docker-compose.yml logs wp-php
   ```

3. **Verify secrets are accessible:**
   ```bash
   docker compose -f srcs/docker-compose.yml exec wp-php ls -la /run/secrets
   ```

### Permission Issues

If you encounter permission errors:

```bash
# Fix WordPress directory permissions
sudo chown -R $USER:$USER /home/jazevedo/data/wordpress
sudo chmod -R 755 /home/jazevedo/data/wordpress

# Fix MariaDB directory permissions
sudo chown -R $USER:$USER /home/jazevedo/data/mariadb
```

### Complete Reset

To start completely fresh:

```bash
cd inception
make fclean
rm -rf /home/jazevedo/data/wordpress/*
rm -rf /home/jazevedo/data/mariadb/*
make
```

## Project Architecture

### Container Communication

```
Internet
   ↓
NGINX (Port 443) ←→ Docker Network (inception) ←→ WordPress (php-fpm:9000)
                                                      ↓
                                                  MariaDB (3306)
```

- **NGINX** is the only entry point (port 443)
- **WordPress** and **MariaDB** are not directly accessible from the host
- All communication happens through the Docker network

### Service Dependencies

```
MariaDB (no dependencies)
   ↓
WordPress (depends on MariaDB)
   ↓
NGINX (depends on WordPress and MariaDB)
```

Docker Compose handles these dependencies automatically via `depends_on`.

---

**For end-user instructions, see USER_DOC.md**
