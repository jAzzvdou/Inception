*This project has been created as part of the 42 curriculum by jazevedo.*

# Inception - Docker Orchestration Project

## Description

This project is a system administration exercise that demonstrates the use of Docker and Docker Compose to orchestrate a multi-container infrastructure. The goal is to set up a complete WordPress website stack using containerized services, following best practices for containerization, security, and data persistence.

The infrastructure consists of three main services:
- **NGINX**: A reverse proxy and web server with TLS/SSL encryption (TLSv1.2/TLSv1.3)
- **WordPress + php-fpm**: A content management system with PHP-FPM for processing PHP requests
- **MariaDB**: A relational database management system for storing WordPress data

All services are containerized using custom Dockerfiles built from Debian 12.11 (penultimate stable version), communicate through a Docker network, and use named volumes for data persistence. The project emphasizes security by using Docker secrets for sensitive information and environment variables for configuration.

## Instructions

### Prerequisites

- A Linux virtual machine
- Docker and Docker Compose v2 installed
- Root or sudo access for initial setup
- Basic knowledge of Docker and containerization concepts

### Troubleshooting: Docker Permissions

If you need to use `sudo` to run `docker` commands or `make`, it means your user is not in the `docker` group. After running the setup script, you need to apply the group changes:

**Option 1 (Recommended):** Apply the group in your current session:
```bash
newgrp docker
```

**Option 2:** Log out and log back in to apply the group changes permanently.

After applying the group, verify it works:
```bash
docker ps  # Should work without sudo
cd inception
make       # Should work without sudo
```

**Why this happens:** The Docker daemon socket (`/var/run/docker.sock`) is owned by the `docker` group. Users need to be in this group to access Docker without `sudo`. The setup script adds your user to the group, but Linux requires a new session (logout/login or `newgrp`) for group changes to take effect.

### Quick Start

1. **Navigate to the project directory:**
   ```bash
   cd inception
   ```

2. **Configure your environment:**
   - Edit `srcs/.env` to set your domain name (e.g., `jazevedo.42.fr`)
   - Update secrets in `secrets/` directory with your passwords
   - Ensure `/etc/hosts` points your domain to your local IP address:
     ```bash
     sudo echo "127.0.0.1 jazevedo.42.fr" >> /etc/hosts
     ```

3. **Build and start the infrastructure:**
   ```bash
   make
   ```

4. **Access your website:**
   - Open `https://jazevedo.42.fr` in your browser
   - Accept the self-signed certificate warning (if applicable)
   - WordPress should be pre-configured and ready to use

### Available Makefile Commands

- `make` or `make inception`: Build and start all containers
- `make clean`: Stop and remove all containers
- `make fclean`: Stop containers, remove images, volumes, and networks
- `make re`: Clean everything and rebuild from scratch

### Stopping the Infrastructure

```bash
cd inception
make clean
```

### Complete Cleanup

```bash
cd inception
make fclean
```

## Resources

### Documentation and References

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [NGINX Documentation](https://nginx.org/en/docs/)
- [WordPress Documentation](https://wordpress.org/support/)
- [MariaDB Documentation](https://mariadb.com/kb/en/documentation/)
- [PHP-FPM Documentation](https://www.php.net/manual/en/install.fpm.php)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

### AI Usage

AI tools were used in the following aspects of this project:

- **Code Review and Optimization**: AI was used to review Dockerfile configurations, identify potential security issues, and suggest improvements for container best practices.
- **Documentation Assistance**: AI helped structure and format documentation files, ensuring clarity and completeness while maintaining technical accuracy.
- **Troubleshooting**: AI assisted in diagnosing and resolving container networking issues and volume mounting problems during development.
- **Configuration Validation**: AI was used to verify that configurations comply with project requirements, such as TLS protocol versions and security constraints.

All AI-generated content was thoroughly reviewed, tested, and understood before integration into the project.

## Project Description

### Docker Usage

This project leverages Docker to containerize each service independently, allowing for isolated, reproducible, and scalable deployments. Each service runs in its own container with a dedicated Dockerfile, ensuring consistency across different environments.

**Key Docker Features Used:**
- **Docker Compose**: Orchestrates multiple containers, manages dependencies, and simplifies deployment
- **Docker Networks**: Enables secure communication between containers without exposing services to the host
- **Docker Volumes**: Provides persistent storage for database and WordPress files
- **Docker Secrets**: Securely manages sensitive information like passwords and credentials
- **Multi-stage builds**: Optimizes image sizes and build processes

### Source Files Structure

The project follows a structured organization:

```
inception/
├── Makefile                 # Main orchestration file
├── secrets/                 # Docker secrets (passwords, credentials)
│   ├── db_root_password.txt
│   ├── wp_db_password.txt
│   ├── wp_admin_password.txt
│   └── wp_user2_password.txt
└── srcs/
    ├── docker-compose.yml   # Service definitions and orchestration
    ├── .env                 # Environment variables
    └── requirements/
        ├── nginx/           # NGINX service
        │   ├── Dockerfile
        │   └── conf/
        ├── wordpress/       # WordPress + php-fpm service
        │   ├── Dockerfile
        │   ├── conf/
        │   └── tools/
        └── mariadb/         # MariaDB service
            ├── Dockerfile
            ├── conf/
            └── tools/
```

### Main Design Choices

1. **Base Image**: Debian 12.11 (penultimate stable version) was chosen for stability and package availability
2. **Service Isolation**: Each service runs in its own container for better security and maintainability
3. **Network Isolation**: Custom Docker network prevents direct host access to database and WordPress services
4. **Volume Management**: Named volumes with bind mounts to `/home/jazevedo/data/` ensure data persistence
5. **Security**: Docker secrets and environment variables prevent credential exposure in code
6. **Process Management**: Services run as proper daemons (nginx, php-fpm, mysqld) without hacky workarounds

### Comparisons

#### Virtual Machines vs Docker

**Virtual Machines:**
- Full operating system with its own kernel
- Higher resource overhead (CPU, RAM, disk)
- Slower startup times (minutes)
- Complete isolation but heavier
- Requires hypervisor software

**Docker:**
- Containerization using host OS kernel
- Lower resource overhead (shares kernel)
- Fast startup times (seconds)
- Process-level isolation, lighter weight
- Native support on Linux

**Why Docker for this project:**
Docker provides the perfect balance between isolation and efficiency for this multi-service application. Each service can be developed, tested, and deployed independently while sharing the host's kernel, resulting in faster builds, easier scaling, and simpler dependency management.

#### Secrets vs Environment Variables

**Environment Variables:**
- Stored in `.env` files or passed directly
- Visible in container environment
- Suitable for non-sensitive configuration
- Easy to modify and version control (for non-sensitive data)
- Accessible via `docker inspect` or process lists

**Docker Secrets:**
- Stored in files mounted as read-only
- Not visible in environment variables
- Ideal for passwords, API keys, certificates
- More secure, not exposed in process lists
- Managed separately from application code

**Why both in this project:**
Environment variables handle non-sensitive configuration (domain names, service names, URLs), while Docker secrets protect critical credentials (database passwords, admin passwords). This separation follows security best practices and prevents accidental credential exposure.

#### Docker Network vs Host Network

**Host Network:**
- Containers use host's network stack directly
- Services accessible on host's IP and ports
- No network isolation
- Simpler but less secure
- Port conflicts possible

**Docker Network:**
- Isolated virtual network for containers
- Services communicate via container names
- Network-level isolation and security
- Port mapping controlled explicitly
- Better for multi-container applications

**Why Docker Network for this project:**
A custom Docker network (`inception`) ensures that MariaDB and WordPress are only accessible through NGINX, not directly from the host. This follows the principle of least privilege and creates a proper reverse proxy architecture where NGINX is the only entry point.

#### Docker Volumes vs Bind Mounts

**Bind Mounts:**
- Direct mapping to host filesystem path
- Immediate file visibility on host
- Easier for development and debugging
- Tied to specific host paths
- Can cause permission issues

**Docker Named Volumes:**
- Managed by Docker
- Can use different drivers (local, NFS, etc.)
- Better portability across systems
- Docker handles permissions
- Can be backed by bind mounts (as in this project)

**Why Named Volumes with Bind Mount Backend:**
This project uses named volumes backed by bind mounts to `/home/jazevedo/data/`. This approach provides:
- Docker volume management benefits (naming, lifecycle)
- Direct host access for backup and inspection
- Data persistence across container restarts
- Compliance with project requirements (data in `/home/login/data/`)

---

**End of README**
