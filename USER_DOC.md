# User Documentation - Inception Project

This document provides clear and simple instructions for end users and administrators on how to use the Inception Docker infrastructure.

## Services Provided by the Stack

The infrastructure provides the following services:

1. **WordPress Website**: A fully functional WordPress website accessible via HTTPS
2. **WordPress Administration Panel**: Access to the WordPress admin dashboard for content management
3. **MariaDB Database**: A MySQL-compatible database storing all WordPress data
4. **NGINX Web Server**: A reverse proxy handling HTTPS connections and routing requests

All services are containerized and run automatically when the stack is started.

## Starting the Project

1. **Navigate to the project directory:**
   ```bash
   cd inception
   ```

2. **Start all services:**
   ```bash
   make
   ```
   
   This command will:
   - Build Docker images for all services (if not already built)
   - Create and start all containers
   - Set up the network and volumes
   - Configure WordPress automatically

3. **Wait for services to be ready:**
   - The first startup may take 1-2 minutes as WordPress is being configured
   - You can check container status with: `docker compose -f srcs/docker-compose.yml ps`

4. **Verify services are running:**
   ```bash
   docker compose -f srcs/docker-compose.yml ps
   ```
   
   You should see three containers running:
   - `nginx` (port 443)
   - `wp-php` (WordPress + php-fpm)
   - `mariadb` (database)

## Stopping the Project

To stop all services:

```bash
cd inception
make clean
```

This will stop and remove all containers while preserving your data in volumes.

## Accessing the Website

1. **Ensure your domain is configured:**
   - The domain should be `jazevedo.42.fr` (replace `jazevedo` with your login)
   - Add this line to `/etc/hosts` if not already present:
     ```bash
     sudo echo "127.0.0.1 jazevedo.42.fr" >> /etc/hosts
     ```

2. **Open your web browser** and navigate to:
   ```
   https://jazevedo.42.fr
   ```

3. **Accept the SSL certificate warning:**
   - Since we use a self-signed certificate, your browser will show a security warning
   - Click "Advanced" and then "Proceed to jazevedo.42.fr" (or similar)
   - This is normal and safe for local development

4. **You should see your WordPress website** - it should be pre-configured and ready to use (no installation page).

## Accessing the Administration Panel

1. **Navigate to the WordPress login page:**
   ```
   https://jazevedo.42.fr/wp-admin
   ```

2. **Login credentials:**
   - The administrator username is stored in the `.env` file (variable `WP_ADMIN_USER`)
   - The administrator password is stored in `inception/secrets/wp_admin_password.txt`
   - To view the password:
     ```bash
     cat inception/secrets/wp_admin_password.txt
     ```

3. **Important**: The administrator username **cannot contain** "admin" or "Administrator" (as per project requirements).

4. **After logging in**, you'll have access to:
   - WordPress Dashboard
   - Posts and Pages management
   - Media library
   - User management
   - Themes and Plugins
   - Settings and Configuration

## Managing Credentials

All sensitive credentials are stored in the `inception/secrets/` directory:

- **`db_root_password.txt`**: MariaDB root user password
- **`wp_db_password.txt`**: WordPress database user password
- **`wp_admin_password.txt`**: WordPress administrator password
- **`wp_user2_password.txt`**: Secondary WordPress user password

### Viewing Credentials

To view a credential:
```bash
cat inception/secrets/<filename>
```

### Changing Credentials

⚠️ **Warning**: Changing credentials after the initial setup may require reconfiguration:

1. **Edit the secret file:**
   ```bash
   nano inception/secrets/<filename>
   ```

2. **If changing database passwords**, you may need to:
   - Update the `.env` file if needed
   - Rebuild containers: `cd inception && make re`

3. **If changing WordPress admin password**, you can:
   - Change it from the WordPress admin panel (Users → Your Profile)
   - Or update the secret file and rebuild

## Checking Service Status

### Check if Services are Running

```bash
cd inception
docker compose -f srcs/docker-compose.yml ps
```

All services should show "Up" status.

### View Service Logs

**View all logs:**
```bash
docker compose -f srcs/docker-compose.yml logs
```

**View logs for a specific service:**
```bash
docker compose -f srcs/docker-compose.yml logs nginx
docker compose -f srcs/docker-compose.yml logs wp-php
docker compose -f srcs/docker-compose.yml logs mariadb
```

**Follow logs in real-time:**
```bash
docker compose -f srcs/docker-compose.yml logs -f
```

### Verify Website Accessibility

1. **Test HTTPS connection:**
   ```bash
   curl -k https://jazevedo.42.fr
   ```
   The `-k` flag ignores SSL certificate verification (needed for self-signed certs)

2. **Check if port 443 is listening:**
   ```bash
   sudo netstat -tlnp | grep 443
   ```
   or
   ```bash
   sudo ss -tlnp | grep 443
   ```

3. **Test from browser:**
   - Open `https://jazevedo.42.fr`
   - You should see the WordPress homepage
   - If you see an error, check the logs

### Verify Data Persistence

1. **Make a change on WordPress** (e.g., create a post or edit a page)

2. **Restart the stack:**
   ```bash
   cd inception
   make clean
   make
   ```

3. **Verify your changes are still there** - they should persist because data is stored in volumes

### Common Issues and Solutions

**Issue: Cannot access website**
- Check if containers are running: `docker compose -f srcs/docker-compose.yml ps`
- Verify `/etc/hosts` has the domain entry
- Check NGINX logs: `docker compose -f srcs/docker-compose.yml logs nginx`

**Issue: WordPress installation page appears**
- WordPress should be pre-configured. If you see the installation page, check:
  - Database connection (check MariaDB logs)
  - WordPress container logs for errors

**Issue: SSL certificate error**
- This is normal with self-signed certificates
- Accept the warning in your browser
- For production, use a proper SSL certificate from a CA

**Issue: Services won't start**
- Check Docker is running: `docker ps`
- Check for port conflicts: `sudo netstat -tlnp | grep 443`
- View error logs: `docker compose -f srcs/docker-compose.yml logs`

## Data Location

Your WordPress data is stored in:
- **WordPress files**: `/home/jazevedo/data/wordpress/`
- **Database files**: `/home/jazevedo/data/mariadb/`

These directories persist even when containers are stopped, ensuring your data is safe.

---

**For developer-specific instructions, see DEV_DOC.md**
