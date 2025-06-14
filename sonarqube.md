# SonarQube Complete Guide

## 1. Core Concepts

### 1.1 Quality Gates
- Reliability (Bugs)
- Security (Vulnerabilities)
- Maintainability (Code Smells)
- Coverage (Test Coverage)
- Duplications (Duplicate Code)
- Size (Lines of Code)

### 1.2 Issue Types
1. Bugs
   - Reliability Issues
   - Runtime Behavior
   - Incorrect Logic

2. Vulnerabilities
   - Security Issues
   - Injection Flaws
   - Authentication Problems

3. Code Smells
   - Maintainability Issues
   - Design Problems
   - Style Violations

4. Security Hotspots
   - Security Review
   - Best Practices
   - Potential Vulnerabilities

## 2. Installation and Setup

### 2.1 Docker Installation
```yaml
version: "3"
services:
  sonarqube:
    image: sonarqube:latest
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    networks:
      - sonarnet
    depends_on:
      - db

  db:
    image: postgres:12
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data
    networks:
      - sonarnet

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:

networks:
  sonarnet:
    driver: bridge
```

### 2.2 System Configuration
```properties
# sonar.properties
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
sonar.web.host=0.0.0.0
sonar.web.port=9000
sonar.search.javaAdditionalOpts=-Xms512m -Xmx512m
sonar.ce.javaOpts=-Xms512m -Xmx2048m -XX:+HeapDumpOnOutOfMemoryError
sonar.web.javaOpts=-Xms512m -Xmx2048m -XX:+HeapDumpOnOutOfMemoryError
```

## 3. Project Configuration

### 3.1 Maven Integration
```xml
<plugin>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>3.9.1</version>
    <executions>
        <execution>
            <phase>verify</phase>
            <goals>
                <goal>sonar</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <sonar.host.url>http://localhost:9000</sonar.host.url>
        <sonar.login>${SONAR_TOKEN}</sonar.login>
        <sonar.coverage.jacoco.xmlReportPaths>
            ${project.build.directory}/site/jacoco/jacoco.xml
        </sonar.coverage.jacoco.xmlReportPaths>
    </configuration>
</plugin>
```

### 3.2 Gradle Integration
```groovy
plugins {
    id "org.sonarqube" version "3.3"
}

sonarqube {
    properties {
        property "sonar.projectKey", "my-project"
        property "sonar.projectName", "My Project"
        property "sonar.sourceEncoding", "UTF-8"
        property "sonar.sources", "src/main"
        property "sonar.tests", "src/test"
        property "sonar.java.binaries", "${buildDir}/classes"
        property "sonar.coverage.jacoco.xmlReportPaths", "${buildDir}/reports/jacoco/test/jacocoTestReport.xml"
    }
}
```

## 4. Quality Profiles

### 4.1 Java Quality Profile
```xml
<profile>
    <name>Company Java Rules</name>
    <language>java</language>
    <rules>
        <rule>
            <repositoryKey>java</repositoryKey>
            <key>S1234</key>
            <priority>MAJOR</priority>
            <parameters>
                <parameter>
                    <key>maximum</key>
                    <value>30</value>
                </parameter>
            </parameters>
        </rule>
    </rules>
</profile>
```

### 4.2 Custom Rules
```java
@Rule(key = "CustomRule")
public class CustomRule extends BaseTreeVisitor implements JavaFileScanner {
    private JavaFileScannerContext context;
    
    @Override
    public void scanFile(JavaFileScannerContext context) {
        this.context = context;
        scan(context.getTree());
    }
    
    @Override
    public void visitMethodInvocation(MethodInvocationTree tree) {
        if (isProhibitedMethod(tree)) {
            context.reportIssue(this, tree, "Avoid using this method");
        }
        super.visitMethodInvocation(tree);
    }
    
    private boolean isProhibitedMethod(MethodInvocationTree tree) {
        return "prohibitedMethod".equals(tree.methodSelect().name());
    }
}
```

## 5. Quality Gates

### 5.1 Default Quality Gate
```json
{
  "name": "Company Quality Gate",
  "conditions": [
    {
      "metric": "new_reliability_rating",
      "op": "GT",
      "error": "1"
    },
    {
      "metric": "new_security_rating",
      "op": "GT",
      "error": "1"
    },
    {
      "metric": "new_maintainability_rating",
      "op": "GT",
      "error": "1"
    },
    {
      "metric": "new_coverage",
      "op": "LT",
      "error": "80"
    },
    {
      "metric": "new_duplicated_lines_density",
      "op": "GT",
      "error": "3"
    }
  ]
}
```

### 5.2 Custom Metrics
```java
@Rule(key = "CustomMetric")
public class CustomMetric extends BaseMetricsSensor {
    @Override
    public void describe(SensorDescriptor descriptor) {
        descriptor.name("Custom Metric");
    }
    
    @Override
    public void execute(SensorContext context) {
        // Calculate metric value
        double metricValue = calculateMetric();
        
        // Save metric
        context.<Double>newMeasure()
            .forMetric(CoreMetrics.CUSTOM_METRIC)
            .on(context.module())
            .withValue(metricValue)
            .save();
    }
}
```

## 6. Integration Examples

### 6.1 Jenkins Pipeline Integration
```groovy
pipeline {
    agent any
    
    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=my-project \
                            -Dsonar.host.url=http://sonarqube:9000 \
                            -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    post {
        always {
            recordIssues enabledForFailure: true, tool: sonarQube()
        }
    }
}
```

### 6.2 GitLab CI Integration
```yaml
sonarqube-check:
  image: maven:3.8.1-jdk-11
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
    GIT_DEPTH: "0"
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script:
    - mvn verify sonar:sonar
  allow_failure: true
  only:
    - merge_requests
    - master
    - develop
```

## 7. Security Configuration

### 7.1 Authentication
```properties
# LDAP Configuration
sonar.security.realm=LDAP
ldap.url=ldap://ldap.example.com:389
ldap.bindDn=cn=sonar,dc=example,dc=org
ldap.bindPassword=secret
ldap.user.baseDn=ou=users,dc=example,dc=org
ldap.user.request=(&(objectClass=inetOrgPerson)(uid={login}))
ldap.user.realNameAttribute=cn
ldap.user.emailAttribute=mail

# SSO Configuration
sonar.auth.github.enabled=true
sonar.auth.github.clientId.secured=your_client_id
sonar.auth.github.clientSecret.secured=your_client_secret
sonar.auth.github.allowUsersToSignUp=true
```

### 7.2 Authorization
```json
{
  "permissions": {
    "groups": [
      {
        "name": "developers",
        "permissions": ["user", "codeviewer", "scan"]
      },
      {
        "name": "administrators",
        "permissions": ["admin"]
      }
    ],
    "users": [
      {
        "login": "jenkins",
        "permissions": ["scan"]
      }
    ]
  }
}
```

## 8. Maintenance

### 8.1 Database Maintenance
```sql
-- Clean up old analysis
DELETE FROM project_measures 
WHERE analysis_uuid NOT IN (
    SELECT uuid 
    FROM snapshots 
    WHERE created_at > now() - interval '90 days'
);

-- Optimize tables
VACUUM ANALYZE project_measures;
VACUUM ANALYZE issues;
```

### 8.2 Backup Strategy
```bash
#!/bin/bash
# Backup script

# Stop SonarQube
docker-compose stop sonarqube

# Backup database
pg_dump -h localhost -U sonar sonar > sonar_$(date +%Y%m%d).sql

# Backup configuration
tar -czf sonar_conf_$(date +%Y%m%d).tar.gz \
    /opt/sonarqube/conf/* \
    /opt/sonarqube/extensions/*

# Start SonarQube
docker-compose start sonarqube

# Clean old backups
find /backup -name "sonar_*.sql" -mtime +7 -delete
find /backup -name "sonar_conf_*.tar.gz" -mtime +7 -delete
```

## 9. Monitoring

### 9.1 System Monitoring
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'sonarqube'
    metrics_path: '/api/monitoring/metrics'
    static_configs:
      - targets: ['sonarqube:9000']

# grafana_dashboard.json
{
  "dashboard": {
    "panels": [
      {
        "title": "Active Users",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "sonarqube_active_users"
          }
        ]
      },
      {
        "title": "Analysis Time",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "sonarqube_analysis_time_seconds"
          }
        ]
      }
    ]
  }
}
```

### 9.2 Performance Tuning
```properties
# Memory Configuration
sonar.ce.javaOpts=-Xms512m -Xmx2048m -XX:+HeapDumpOnOutOfMemoryError
sonar.web.javaOpts=-Xms512m -Xmx2048m -XX:+HeapDumpOnOutOfMemoryError
sonar.search.javaOpts=-Xms512m -Xmx512m

# Thread Configuration
sonar.ce.workerCount=4
sonar.web.http.minThreads=5
sonar.web.http.maxThreads=50

# Cache Configuration
sonar.search.javaAdditionalOpts=-Xms512m -Xmx512m
sonar.search.searchTimeout=300
```

## 10. Troubleshooting

### 10.1 Common Issues
```bash
# Check logs
docker logs sonarqube

# Check system status
curl -u admin:admin http://localhost:9000/api/system/status

# Verify database connection
curl -u admin:admin http://localhost:9000/api/system/db_migration_status

# Check compute engine status
curl -u admin:admin http://localhost:9000/api/ce/activity
```

### 10.2 Debug Mode
```properties
# Enable debug logging
sonar.log.level=DEBUG
sonar.ce.logging.logsRetentionDays=30
sonar.web.accessLogs.enable=true

# Enable JMX monitoring
sonar.ce.javaOpts=-Dcom.sun.management.jmxremote \
    -Dcom.sun.management.jmxremote.port=9010 \
    -Dcom.sun.management.jmxremote.authenticate=false \
    -Dcom.sun.management.jmxremote.ssl=false
``` 
