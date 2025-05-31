# Maven Complete Guide

## 1. Core Concepts

### 1.1 Project Object Model (POM)
- Project metadata
- Dependencies
- Build configuration
- Project relationships
- Inheritance
- Aggregation

### 1.2 Build Lifecycle
1. validate  # Validate project structure
2. compile   # Compile source code
3. test      # Run unit tests
4. package   # Create JAR/WAR
5. verify    # Run integration tests
6. install   # Install to local repo
7. deploy    # Deploy to remote repo

## 2. Project Structure
```
project-root/
├── src/
│   ├── main/
│   │   ├── java/        # Source code
│   │   ├── resources/   # Resources
│   │   └── webapp/      # Web resources
│   └── test/
│       ├── java/        # Test code
│       └── resources/   # Test resources
├── target/              # Compiled code
└── pom.xml             # Project configuration
```

## 3. Dependency Management

### 3.1 Basic Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.7.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.22</version>
        <scope>provided</scope>
    </dependency>
    
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 3.2 Dependency Management
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.7.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 3.3 Dependency Scopes
- compile  # Default, available everywhere
- provided # Available at compile/test
- runtime  # Available at runtime/test
- test     # Available only for testing
- system   # System path dependencies
- import   # Import dependency management

## 4. Build Configuration

### 4.1 Plugins
```xml
<build>
    <plugins>
        <!-- Compiler plugin -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
        
        <!-- Spring Boot plugin -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.7.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        
        <!-- Test coverage -->
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.7</version>
            <executions>
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 4.2 Resources
```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

## 5. Profiles

### 5.1 Profile Types
```xml
<!-- Environment-specific profiles -->
<profiles>
    <profile>
        <id>development</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <db.url>jdbc:mysql://localhost:3306/devdb</db.url>
            <db.username>devuser</db.username>
        </properties>
    </profile>
    
    <profile>
        <id>production</id>
        <properties>
            <db.url>jdbc:mysql://prod-server:3306/proddb</db.url>
            <db.username>produser</db.username>
        </properties>
    </profile>
</profiles>
```

### 5.2 Profile Activation
```xml
<activation>
    <!-- By property -->
    <property>
        <name>env</name>
        <value>prod</value>
    </property>
    
    <!-- By OS -->
    <os>
        <name>Linux</name>
        <family>unix</family>
    </os>
    
    <!-- By file presence -->
    <file>
        <exists>${basedir}/src/main/resources/config.properties</exists>
    </file>
</activation>
```

## 6. Multi-Module Projects

### 6.1 Parent POM
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>common</module>
        <module>service</module>
        <module>web</module>
    </modules>
    
    <properties>
        <java.version>11</java.version>
        <spring.version>5.3.20</spring.version>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <!-- Common dependencies -->
        </dependencies>
    </dependencyManagement>
    
    <build>
        <pluginManagement>
            <!-- Common plugins -->
        </pluginManagement>
    </build>
</project>
```

### 6.2 Child Module
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

## 7. Real-World Examples

### 7.1 Spring Boot Application
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>spring-boot-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    
    <properties>
        <java.version>11</java.version>
        <lombok.version>1.18.22</lombok.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        
        <!-- Utilities -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    
    <profiles>
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <spring.profiles.active>dev</spring.profiles.active>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <spring.profiles.active>prod</spring.profiles.active>
            </properties>
        </profile>
    </profiles>
</project>
```

### 7.2 Microservice Project
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>microservice-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>api</module>
        <module>service</module>
        <module>repository</module>
    </modules>
    
    <properties>
        <java.version>11</java.version>
        <spring-cloud.version>2021.0.3</spring-cloud.version>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.7</version>
            </plugin>
        </plugins>
    </build>
</project>
```

## 8. Best Practices

### 8.1 Dependency Management
1. Use dependencyManagement
2. Specify versions explicitly
3. Regular dependency updates
4. Use BOM (Bill of Materials)
5. Minimize transitive dependencies

### 8.2 Build Configuration
1. Use plugin management
2. Configure in parent POM
3. Standardize versions
4. Use properties for versions

### 8.3 Project Structure
1. Logical module separation
2. Consistent naming
3. Clear dependencies
4. Proper resource management

## 9. Common Commands
```bash
# Basic commands
mvn clean           # Clean target directory
mvn compile         # Compile source code
mvn test           # Run unit tests
mvn package        # Create JAR/WAR
mvn install        # Install to local repo
mvn deploy         # Deploy to remote repo

# Dependency analysis
mvn dependency:tree # Show dependency tree
mvn dependency:analyze # Analyze dependencies
mvn versions:display-dependency-updates # Check for updates

# Running with profiles
mvn clean install -P prod # Run with production profile

# Skip tests
mvn clean install -DskipTests # Skip running tests
mvn clean install -Dmaven.test.skip=true # Skip compilation and running

# Debug
mvn -X clean install # Debug mode
mvn help:effective-pom # Show effective POM
```

## 10. Integration with CI/CD

### 10.1 Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.1'
        jdk 'JDK 11'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Code Quality') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                sh 'mvn deploy'
            }
        }
    }
} 
