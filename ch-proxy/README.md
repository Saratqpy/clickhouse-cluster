```markdown
# Docker Compose Configuration Documentation

This configuration file is for setting up the `ch-proxy` service using Docker Compose. The `ch-proxy` service is configured as a proxy for interacting with various systems.

## File Structure

### Version

```yaml
version: '3.8'
```

### Services

#### ch-proxy

- **Image:** `contentsquareplatform/chproxy:v1.26.4`
  - The Docker image version used for the `ch-proxy` service.
- **Platform:** `linux/amd64`
  - The platform used for running the container.
- **Container Name:** `ch-proxy`
- **Hostname:** `ch-proxy`
- **Networks:**
  - **Network Name:** `cluster_1S_2R_ch_proxy`
  - **Static IP Address:** `172.23.0.8`
- **Ports:**
  - `127.0.0.1:443:443` - Host port 443 is mapped to container port 443.
  - `127.0.0.1:80:80` - Host port 80 is mapped to container port 80.
- **Volumes:**
  - `./config/config.yml:/opt/config.yml`
    - The local configuration file `config.yml` is mapped to `/opt/config.yml` in the container.
- **Command:**
  - `[-config, /opt/config.yml]`
    - Command to run the container using the configuration file.

### Networks

#### cluster_1S_2R_ch_proxy

- **Network Name:** `cluster_1S_2R_ch_proxy`
- **IPAM Configuration:**
  - **Subnet:** `172.23.0.0/24`

#### clickhouse_network

- **Driver:** `bridge`
- **Network Name:** `clickhouse_network`
- **External:** `true`
  - This network is created externally and used as-is.

## Usage Instructions

1. **Starting the Containers:**

   Use the following command to start the services:
   ```bash
   docker-compose up -d
   ```

2. **Stopping the Containers:**

   Use the following command to stop the services:
   ```bash
   docker-compose down
   ```

3. **Checking Container Status:**

   To check the status of the containers, use:
   ```bash
   docker-compose ps
   ```

## Additional Notes

- Ensure that the `config.yml` file is present in the `./config/` directory and is properly configured.
