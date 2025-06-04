// ...existing code...

### CI/CD Pipeline Commands
- `docker build --no-cache` - Force fresh build without cache
- `docker build --build-arg KEY=value` - Pass build-time variables
- `docker build --target stage` - Build specific stage in multi-stage build
- `docker save -o image.tar image:tag` - Export image to tar file
- `docker load -i image.tar` - Load image from tar file
- `docker history image:tag` - Show image layer history

### Monitoring & Debugging
- `docker events` - Get real-time events
- `docker stats --all` - Monitor all containers
- `docker logs --since 1h` - View logs from last hour
- `docker logs --tail 100` - View last 100 log lines
- `docker logs -f --until 1h` - Follow logs until specified time
- `docker cp container:src dest` - Copy files from container
- `docker diff container` - Show changes to container's filesystem

### Resource Management
- `docker run --memory="1g"` - Limit memory usage
- `docker run --cpus="1.5"` - Limit CPU usage
- `docker run --device=/dev/sda:/dev/xvda` - Map devices
- `docker run --ulimit nofile=1024:1024` - Set ulimits
- `docker update --memory="2g" container` - Update container resources
- `docker run --storage-opt size=10G` - Set storage limits

### Health Checks & Auto-healing
- `docker run --health-cmd="curl -f localhost"` - Define health check
- `docker run --health-interval=5s` - Set check interval
- `docker run --health-retries=3` - Set retry count
- `docker run --health-start-period=30s` - Set initial grace period
- `docker run --restart=always` - Auto restart policy

### Advanced Networking
- `docker network create --subnet=172.18.0.0/16` - Custom subnet
- `docker network create --driver overlay` - Create overlay network
- `docker run --dns=8.8.8.8` - Set custom DNS
- `docker run --add-host host:ip` - Add hosts file entry
- `docker run --mac-address="02:42:ac:11:00:02"` - Set MAC address

### Security & Compliance
- `docker run --security-opt seccomp=profile.json` - Apply seccomp profile
- `docker run --read-only` - Make container filesystem read-only
- `docker run --cap-drop ALL` - Drop all capabilities
- `docker run --pids-limit 100` - Limit number of processes
- `docker run --no-new-privileges` - Disable privilege escalation

### Multi-Container Management
- `docker-compose --compatibility up` - Enable Swarm compatibility
- `docker-compose -f prod.yml up` - Use specific compose file
- `docker-compose --scale service=3` - Scale service instances
- `docker-compose config` - Validate compose file
- `docker-compose push` - Push service images

### Container Orchestration (Swarm)
- `docker stack deploy -c stack.yml app` - Deploy stack
- `docker stack services app` - List stack services
- `docker stack ps app` - List stack tasks
- `docker service update --image new:tag service` - Rolling update
- `docker service rollback service` - Rollback service
- `docker service logs service` - View service logs

### Performance & Optimization
- `docker builder prune` - Clean build cache
- `docker run --runtime=nvidia` - Use specific runtime
- `docker build --squash` - Squash image layers
- `docker stats --format "{{.Container}}: {{.CPUPerc}}"` - Custom stats format
- `docker system events --filter type=container` - Filter events

### Backup & Disaster Recovery
- `docker commit container image:tag` - Create image from container
- `docker export container > backup.tar` - Export container filesystem
- `docker import backup.tar` - Import container filesystem
- `docker run --mount type=volume,src=vol,dst=/data` - Named volume mount
- `docker run --tmpfs /tmp` - Mount tmpfs

### Advanced Logging
- `docker run --log-driver=syslog` - Use syslog driver
- `docker run --log-opt syslog-address=udp://1.2.3.4:1111` - Log options
- `docker run --log-driver json-file --log-opt max-size=10m` - Rotate logs
- `docker logs --details` - Show extra details

### Configuration Management
- `docker config create app-config config.json` - Create config
- `docker config ls` - List configs
- `docker secret create app-secret secret.txt` - Create secret
- `docker secret ls` - List secrets
- `docker service update --secret-rm` - Remove service secret

### Development Workflow
- `docker run --init` - Use init process
- `docker run --rm` - Auto-remove container
- `docker build --target dev .` - Build development stage
- `docker-compose watch` - Watch for changes
- `docker context use remote` - Switch Docker context