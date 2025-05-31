# Nexus Repository Manager Complete Guide

## 1. Core Concepts

### 1.1 Repository Types
1. Maven Repositories
   - Proxy (cache remote repositories)
   - Hosted (internal artifacts)
   - Group (aggregate multiple repositories)

2. Docker Registries
   - Proxy (cache Docker Hub)
   - Hosted (private images)
   - Group (multiple registries)

3. npm Repositories
   - Proxy (cache npmjs.org)
   - Hosted (private packages)
   - Group (multiple sources)

4. Other Formats
   - PyPI (Python packages)
   - Raw (generic files)
   - Yum (RPM packages)
   - Apt (Debian packages)

### 1.2 Storage Configuration
```properties
# Blob Stores
nexus.store.type=file
nexus.store.path=/nexus-data/blobs
nexus.store.quota.enabled=true
nexus.store.quota.limit=1TB

# Repository Storage
repository.maven-central.storage.blobStoreName=default
repository.maven-central.storage.strictContentTypeValidation=true
```

## 2. Installation and Setup

### 2.1 Docker Installation
```yaml
version: '3.8'
services:
  nexus:
    image: sonatype/nexus3:latest
    ports:
      - "8081:8081"
      - "8082:8082"  # Docker registry
      - "8083:8083"  # npm registry
    volumes:
      - nexus-data:/nexus-data
    environment:
      - INSTALL4J_ADD_VM_PARAMS=-Xms2703m -Xmx2703m -XX:MaxDirectMemorySize=2703m
    restart: unless-stopped

volumes:
  nexus-data:
```

### 2.2 Initial Configuration
```bash
# Get initial admin password
docker exec -it nexus cat /nexus-data/admin.password

# Basic security settings
security.anonymousAccess=false
security.realms=\
  NexusAuthenticatingRealm,\
  NexusAuthorizingRealm,\
  DockerToken
```

## 3. Repository Configuration

### 3.1 Maven Repository Setup
```xml
<!-- settings.xml -->
<settings>
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://nexus:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>
  
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>
  
  <profiles>
    <profile>
      <id>nexus</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  
  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
</settings>
```

### 3.2 Docker Registry Configuration
```yaml
# /etc/docker/daemon.json
{
  "insecure-registries": [
    "nexus:8082"
  ],
  "registry-mirrors": [
    "http://nexus:8082"
  ]
}

# Docker client configuration
{
  "auths": {
    "nexus:8082": {
      "auth": "base64_encoded_credentials"
    }
  }
}
```

### 3.3 npm Registry Setup
```bash
# Configure npm client
npm config set registry http://nexus:8081/repository/npm-group/

# Add authentication
npm login --registry=http://nexus:8081/repository/npm-hosted/

# .npmrc file
registry=http://nexus:8081/repository/npm-group/
email=admin@example.com
always-auth=true
_auth=base64_encoded_credentials
```

## 4. Security Configuration

### 4.1 Role-Based Access Control
```json
{
  "roles": [
    {
      "id": "nx-developer",
      "name": "Developer Role",
      "description": "Developer access to repositories",
      "privileges": [
        "nx-repository-view-*-*-read",
        "nx-repository-view-*-*-browse",
        "nx-repository-view-maven2-*-upload"
      ]
    },
    {
      "id": "nx-deployer",
      "name": "Deployer Role",
      "description": "CI/CD deployment access",
      "privileges": [
        "nx-repository-view-*-*-read",
        "nx-repository-view-*-*-browse",
        "nx-repository-view-*-*-add",
        "nx-repository-view-*-*-edit"
      ]
    }
  ]
}
```

### 4.2 User Management
```json
{
  "users": [
    {
      "userId": "developer1",
      "firstName": "John",
      "lastName": "Doe",
      "emailAddress": "john@example.com",
      "password": "encrypted_password",
      "roles": ["nx-developer"]
    },
    {
      "userId": "cicd",
      "firstName": "CI",
      "lastName": "CD",
      "emailAddress": "cicd@example.com",
      "password": "encrypted_password",
      "roles": ["nx-deployer"]
    }
  ]
}
```

### 4.3 SSL Configuration
```properties
# jetty-https.xml
<Configure id="Server" class="org.eclipse.jetty.server.Server">
  <Call name="addConnector">
    <Arg>
      <New class="org.eclipse.jetty.server.ServerConnector">
        <Arg name="server"><Ref refid="Server"/></Arg>
        <Arg name="factories">
          <Array type="org.eclipse.jetty.server.ConnectionFactory">
            <Item>
              <New class="org.eclipse.jetty.server.SslConnectionFactory">
                <Arg name="next">http/1.1</Arg>
                <Arg name="sslContextFactory">
                  <New class="org.eclipse.jetty.util.ssl.SslContextFactory$Server">
                    <Set name="keyStorePath">/nexus-data/etc/ssl/keystore.jks</Set>
                    <Set name="keyStorePassword">password</Set>
                  </New>
                </Arg>
              </New>
            </Item>
          </Array>
        </Arg>
        <Set name="port">8443</Set>
      </New>
    </Arg>
  </Call>
</Configure>
```

## 5. Integration Examples

### 5.1 Maven Project Integration
```xml
<!-- pom.xml -->
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>http://nexus:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://nexus:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

### 5.2 Jenkins Pipeline Integration
```groovy
pipeline {
    agent any
    
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "nexus:8081"
        NEXUS_REPOSITORY = "maven-releases"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
    }
    
    stages {
        stage("Build") {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: filesByGlob[0].path,
                             type: pom.packaging]
                        ]
                    )
                }
            }
        }
    }
}
```

## 6. Maintenance and Monitoring

### 6.1 Cleanup Policies
```json
{
  "name": "maven-cleanup",
  "format": "maven2",
  "mode": "delete",
  "criteria": {
    "lastBlobUpdated": 60,
    "lastDownloaded": 90,
    "releaseType": "releases"
  }
}
```

### 6.2 Scheduled Tasks
```properties
# Task: Compact blob store
task.name=Compact blob store
task.type=blobstore.compact
task.schedule=0 0 2 * * ?

# Task: Cleanup repositories
task.name=Cleanup repositories
task.type=repository.cleanup
task.schedule=0 0 1 * * ?

# Task: Rebuild search index
task.name=Rebuild search index
task.type=rebuild.index
task.schedule=0 0 4 * * ?
```

### 6.3 Monitoring Configuration
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'nexus'
    metrics_path: '/service/metrics/prometheus'
    static_configs:
      - targets: ['nexus:8081']

# grafana_dashboard.json
{
  "dashboard": {
    "panels": [
      {
        "title": "Repository Health",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "nexus_repository_health_status"
          }
        ]
      }
    ]
  }
}
```

## 7. Backup and Recovery

### 7.1 Backup Strategy
```bash
#!/bin/bash
# Backup script

# Stop Nexus
docker stop nexus

# Backup data directory
tar -czf nexus-data-$(date +%Y%m%d).tar.gz /nexus-data

# Backup configuration
cp -r /nexus-data/etc/nexus.properties backup/
cp -r /nexus-data/etc/ssl backup/

# Start Nexus
docker start nexus

# Clean old backups
find /backup -name "nexus-data-*.tar.gz" -mtime +7 -delete
```

### 7.2 Recovery Procedure
```bash
#!/bin/bash
# Recovery script

# Stop Nexus
docker stop nexus

# Restore data
tar -xzf nexus-data-backup.tar.gz -C /

# Restore configuration
cp -r backup/nexus.properties /nexus-data/etc/
cp -r backup/ssl /nexus-data/etc/

# Fix permissions
chown -R 200:200 /nexus-data

# Start Nexus
docker start nexus
```

## 8. Troubleshooting Guide

### 8.1 Common Issues
```bash
# Check Nexus logs
docker logs nexus

# Check system resources
docker stats nexus

# Verify connectivity
curl -u admin:admin123 http://nexus:8081/service/rest/v1/status

# Check repository health
curl -u admin:admin123 http://nexus:8081/service/rest/v1/status/check
```

### 8.2 Performance Tuning
```properties
# JVM Configuration
-Xms2703m
-Xmx2703m
-XX:MaxDirectMemorySize=2703m
-XX:+UnlockExperimentalVMOptions
-XX:+UseCGroupMemoryLimitForHeap

# Garbage Collection
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:InitiatingHeapOccupancyPercent=70
``` 
