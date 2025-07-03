# Solr Docker Installation with Authentication

This repository provides a secure Solr installation using Docker Compose, featuring:
- Basic authentication with password hashing
- Persistent data storage
- Easy configuration

## Prerequisites

1. Docker and Docker Compose installed
2. `pwgen` installed (for password generation)
3. `openssl` installed (for TLS certificate generation)

## Setup Instructions

### 1. Clone the repository
```bash
git clone https://github.com/khalMeg/Solr9.8-docker.git solr-docker
cd solr-docker
```
> Note: You can keep the default configuration with `myuser:mypass`, or generate your own credentials. 

### 2. Generate password hash
1. Make the script executable:
   ```bash
   chmod +x solr-password.sh
   ```
2. Generate a hash for your password:
   ```bash
   ./solr-password.sh "your_secure_password"
   ```
3. The output will be in the format: `<hash> <salt>`

### 3. Configure security.json
1. Edit `security.json`:
   ```json
   {
   "authentication":{
      "blockUnknown": true,
      "class":"solr.BasicAuthPlugin",
      "credentials":{"<YOUR_USER>":"<hash> <salt>"}, // Replace with <YOUR_USER> and the generated hash 
      "realm":"My Solr users"
      "forwardCredentials": false,
      "requireSsl": true
   },
   "authorization":{
      "class":"solr.RuleBasedAuthorizationPlugin",
      "permissions":[
        {"name":"security-edit", "role":"admin"}
      ],
      "user-role":{"<YOUR_USER>":"admin"}  // Replace <YOUR_USER>
   }}

   ```
### 4. Update SOLR_OPTS ENV variable with the your user:pass (plain text) in docker-compose.yml 
   ```yml
   ...
   environment:
     ...
     SOLR_OPTS: "-Dbasicauth=<YOUR_USER>:<YOUR_PASSWORD>"
     ...
   ```
### 5. Start Solr
```bash
docker-compose up -d
```

## Accessing Solr

### Web Interface
- URL: https://localhost:8983
- Username: `<YOUR_USER>` # default user: myuser
- Password: `<YOUR_USER>` # default password: mypass

### API Access
```bash
curl -k -u solradmin:your_secure_password \
  "https://localhost:8983/solr/admin/cores?wt=json"
```

## Environment Variables

You can customize the installation by setting these environment variables in `docker-compose.yml`:

| Variable | Default | Description |
|----------|---------|-------------|
| `SOLR_ADMIN_USER` | solradmin | Admin username |
| `SOLR_ADMIN_PASSWORD` | | Admin password |
| `SOLR_JAVA_MEM` | -Xms512m -Xmx2g | JVM memory settings |
| `SOLR_SSL_KEY_STORE_PASSWORD` | changeit | Keystore password |

## Maintenance

### Stop Solr
```bash
docker-compose down
```

### Start Solr
```bash
docker-compose up -d
```

### View Logs
```bash
docker-compose logs -f solr
```

### Backup Data
```bash
docker-compose exec solr tar czf /tmp/solr-backup.tar.gz /var/solr/data
docker cp solr_server:/tmp/solr-backup.tar.gz .
```

## Contributing
Contributions are welcome! Please open an issue or submit a pull request.
