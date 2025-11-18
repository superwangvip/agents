---
name: maven-build-management
description: Master Maven build automation, dependency management, multi-module projects, and CI/CD integration. Use when setting up Maven projects, optimizing build performance, or implementing enterprise build strategies.
---

# Maven Build Management

Comprehensive guide to Maven build automation, dependency management, multi-module project structures, and enterprise-grade build optimization strategies.

## When to Use This Skill

- Setting up new Maven projects or converting existing projects
- Optimizing build performance and dependency management
- Creating multi-module enterprise applications
- Configuring Maven profiles for different environments
- Implementing CI/CD pipelines with Maven
- Managing build quality gates and code quality checks
- Customizing Maven lifecycle and plugin execution
- Setting up repository management and artifact deployment
- Debugging build issues and dependency conflicts

## Core Concepts

### 1. Maven Project Object Model (POM)
- **Project Coordinates**: groupId, artifactId, version
- **Dependencies**: External libraries and their versions
- **Build Lifecycle**: compile, test, package, install, deploy phases
- **Plugins**: Build tools and their configuration
- **Properties**: Configuration variables and placeholders

### 2. Repository Management
- **Local Repository**: ~/.m2/repository cache
- **Remote Repository**: Maven Central and custom repositories
- **Corporate Repository**: Nexus, Artifactory for internal artifacts
- **SNAPSHOT vs RELEASE**: Development and release versions

### 3. Build Profiles
- **Environment-specific**: dev, test, staging, production
- **Feature-specific**: optional dependencies and configurations
- **Build-specific**: different JDK versions or build tools

## Project Structure Examples

### Single Module Maven Project

```xml
<!-- pom.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!-- Project Coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-application</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>My Application</name>
    <description>Example Maven application</description>

    <!-- Project Properties -->
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <!-- Dependency Versions -->
        <spring.boot.version>3.2.0</spring.boot.version>
        <junit.version>5.10.1</junit.version>
        <mockito.version>5.7.0</mockito.version>
        <lombok.version>1.18.30</lombok.version>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
    </properties>

    <!-- Dependency Management -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- Dependencies -->
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
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Utilities -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Build Configuration -->
    <build>
        <finalName>${project.artifactId}-${project.version}</finalName>

        <!-- Resources -->
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
                <includes>
                    <include>application*.yml</include>
                    <include>application*.properties</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>false</filtering>
                <excludes>
                    <exclude>application*.yml</exclude>
                    <exclude>application*.properties</exclude>
                </excludes>
            </resource>
        </resources>

        <!-- Plugin Management -->
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.11.0</version>
                    <configuration>
                        <source>${maven.compiler.source}</source>
                        <target>${maven.compiler.target}</target>
                        <annotationProcessorPaths>
                            <path>
                                <groupId>org.projectlombok</groupId>
                                <artifactId>lombok</artifactId>
                                <version>${lombok.version}</version>
                            </path>
                            <path>
                                <groupId>org.mapstruct</groupId>
                                <artifactId>mapstruct-processor</artifactId>
                                <version>${mapstruct.version}</version>
                            </path>
                        </annotationProcessorPaths>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>

        <!-- Plugins -->
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring.boot.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.2</version>
                <configuration>
                    <includes>
                        <include>**/*Test.java</include>
                        <include>**/*Tests.java</include>
                    </includes>
                    <systemPropertyVariables>
                        <spring.profiles.active>test</spring.profiles.active>
                    </systemPropertyVariables>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.11</version>
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

    <!-- Profiles -->
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
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>3.1.0</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```

### Multi-Module Enterprise Project

```xml
<!-- parent/pom.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>enterprise-project</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>Enterprise Project Parent</name>

    <!-- Modules -->
    <modules>
        <module>common</module>
        <module>domain</module>
        <module>infrastructure</module>
        <module>application</module>
        <module>web-api</module>
        <module>batch-processor</module>
    </modules>

    <!-- Properties -->
    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

        <!-- Spring Boot -->
        <spring-boot.version>3.2.0</spring-boot.version>

        <!-- Database -->
        <postgresql.version>42.7.1</postgresql.version>
        <flyway.version>9.22.3</flyway.version>

        <!-- Testing -->
        <junit.version>5.10.1</junit.version>
        <testcontainers.version>1.19.3</testcontainers.version>

        <!-- Code Quality -->
        <spotbugs.version>4.8.2.0</spotbugs.version>
        <checkstyle.version>10.12.5</checkstyle.version>
        <pmd.version>6.55.0</pmd.version>

        <!-- Build Tools -->
        <maven.compiler.plugin.version>3.11.0</maven.compiler.plugin.version>
        <maven.surefire.plugin.version>3.2.2</maven.surefire.plugin.version>
        <maven.failsafe.plugin.version>3.2.2</maven.failsafe.plugin.version>
    </properties>

    <!-- Dependency Management -->
    <dependencyManagement>
        <dependencies>
            <!-- Spring Boot BOM -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!-- Testcontainers BOM -->
            <dependency>
                <groupId>org.testcontainers</groupId>
                <artifactId>testcontainers-bom</artifactId>
                <version>${testcontainers.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!-- Internal Modules -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>common</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>domain</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>infrastructure</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>application</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- Common Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Plugin Management -->
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven.compiler.plugin.version}</version>
                <configuration>
                    <release>${maven.compiler.release}</release>
                    <parameters>true</parameters>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven.surefire.plugin.version}</version>
                <configuration>
                    <parallel>methods</parallel>
                    <threadCount>4</threadCount>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>${maven.failsafe.plugin.version}</version>
            </plugin>

            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.11</version>
            </plugin>

            <plugin>
                <groupId>com.github.spotbugs</groupId>
                <artifactId>spotbugs-maven-plugin</artifactId>
                <version>${spotbugs.version}</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>${checkstyle.version}</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-pmd-plugin</artifactId>
                <version>${pmd.version}</version>
            </plugin>
        </plugins>
    </pluginManagement>

    <!-- Build Configuration -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>3.4.1</version>
                <executions>
                    <execution>
                        <id>enforce-maven</id>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                        <configuration>
                            <rules>
                                <requireMavenVersion>
                                    <version>[3.9.0,)</version>
                                </requireMavenVersion>
                                <requireJavaVersion>
                                    <version>[17,)</version>
                                </requireJavaVersion>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <groupId>com.github.spotbugs</groupId>
                <artifactId>spotbugs-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <effort>Max</effort>
                    <threshold>Low</threshold>
                    <failOnError>true</failOnError>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <!-- Reporting -->
    <reporting>
        <plugins>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <reportSets>
                    <reportSet>
                        <reports>
                            <report>report</report>
                        </reports>
                    </reportSet>
                </reportSets>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-project-info-reports-plugin</artifactId>
                <version>3.5.0</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-report-plugin</artifactId>
                <version>3.2.2</version>
            </plugin>
        </plugins>
    </reporting>
</project>
```

## Advanced Maven Patterns

### Custom Maven Plugin

```java
// Custom Maven Plugin
@Mojo(name = "generate-code", defaultPhase = LifecyclePhase.GENERATE_SOURCES)
public class CodeGeneratorMojo extends AbstractMojo {

    @Parameter(property = "outputDirectory", defaultValue = "${project.build.directory}/generated-sources/custom")
    private File outputDirectory;

    @Parameter(property = "packageName", required = true)
    private String packageName;

    @Parameter(property = "className", required = true)
    private String className;

    @Override
    public void execute() throws MojoExecutionException {
        getLog().info("Generating code for class: " + className);

        try {
            generateCode();
            getLog().info("Code generation completed successfully");
        } catch (Exception e) {
            throw new MojoExecutionException("Code generation failed", e);
        }
    }

    private void generateCode() throws IOException {
        if (!outputDirectory.exists()) {
            outputDirectory.mkdirs();
        }

        String code = generateClassCode();
        File outputFile = new File(outputDirectory, className + ".java");

        try (FileWriter writer = new FileWriter(outputFile)) {
            writer.write(code);
        }
    }

    private String generateClassCode() {
        return String.format("""
            package %s;

            public class %s {
                // Generated class
                public static void main(String[] args) {
                    System.out.println("Hello from generated class!");
                }
            }
            """, packageName, className);
    }
}
```

### Maven Assembly Plugin for Custom Distributions

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.6.0</version>
    <configuration>
        <descriptors>
            <descriptor>src/assembly/distribution.xml</descriptor>
        </descriptors>
        <tarLongFileMode>posix</tarLongFileMode>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

```xml
<!-- src/assembly/distribution.xml -->
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3
          http://maven.apache.org/xsd/assembly-1.1.3.xsd">
    <id>distribution</id>
    <formats>
        <format>tar.gz</format>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>

    <fileSets>
        <fileSet>
            <directory>src/main/scripts</directory>
            <outputDirectory>bin</outputDirectory>
            <fileMode>0755</fileMode>
        </fileSet>
        <fileSet>
            <directory>src/main/config</directory>
            <outputDirectory>config</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>${project.build.directory}</directory>
            <outputDirectory>lib</outputDirectory>
            <includes>
                <include>*.jar</include>
            </includes>
        </fileSet>
    </fileSets>

    <dependencySets>
        <dependencySet>
            <outputDirectory>lib</outputDirectory>
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>
</assembly>
```

### Maven Enforcer Plugin for Quality Gates

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>3.4.1</version>
    <executions>
        <execution>
            <id>enforce-quality-standards</id>
            <goals>
                <goal>enforce</goal>
            </goals>
            <configuration>
                <rules>
                    <!-- Require specific Maven and Java versions -->
                    <requireMavenVersion>
                        <version>[3.9.0,)</version>
                    </requireMavenVersion>
                    <requireJavaVersion>
                        <version>[17,)</version>
                    </requireJavaVersion>

                    <!-- No snapshot dependencies in releases -->
                    <requireReleaseDeps>
                        <message>No Snapshots Allowed!</message>
                        <onlyWhenRelease>true</onlyWhenRelease>
                    </requireReleaseDeps>

                    <!-- Enforce dependency convergence -->
                    <dependencyConvergence/>

                    <!-- Ban specific dependencies -->
                    <bannedDependencies>
                        <excludes>
                            <exclude>commons-logging</exclude>
                            <exclude>log4j:log4j</exclude>
                        </excludes>
                    </bannedDependencies>

                    <!-- Require specific properties -->
                    <requireProperty>
                        <property>project.version</property>
                        <message>Project version must be set</message>
                    </requireProperty>

                    <!-- Bytecode compatibility -->
                    <enforceBytecodeVersion>
                        <maxJdkVersion>17</maxJdkVersion>
                    </enforceBytecodeVersion>
                </rules>
                <fail>true</fail>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## Performance Optimization

### Parallel Builds

```bash
# Enable parallel builds
mvn -T 4 clean install  # Use 4 threads
mvn -T 1C clean install # Use 1 thread per CPU core

# Enable parallel test execution
mvn test -Dparallel=methods -DthreadCount=4
```

### Build Time Optimization

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <fork>true</fork>
        <maxmem>1024m</maxmem>
        <compilerArgs>
            <arg>-parameters</arg>
            <arg>-Xlint:all</arg>
        </compilerArgs>
    </configuration>
</plugin>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <parallel>methods</parallel>
        <threadCount>4</threadCount>
        <useSystemClassLoader>false</useSystemClassLoader>
        <reuseForks>true</reuseForks>
        <forkCount>2</forkCount>
    </configuration>
</plugin>
```

### Incremental Builds

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <webResources>
            <resource>
                <filtering>true</filtering>
                <directory>src/main/webapp</directory>
                <includes>
                    <include>**/web.xml</include>
                </includes>
            </resource>
        </webResources>
        <warSourceDirectory>${project.build.directory}/${project.build.finalName}</warSourceDirectory>
        <packagingExcludes>WEB-INF/lib/*.jar</packagingExcludes>
    </configuration>
</plugin>
```

## CI/CD Integration

### GitHub Actions Workflow

```yaml
name: Maven CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: [17, 21]

    steps:
    - uses: actions/checkout@v3

    - name: Set up JDK ${{ matrix.java-version }}
      uses: actions/setup-java@v3
      with:
        java-version: ${{ matrix.java-version }}
        distribution: 'temurin'
        cache: 'maven'

    - name: Run tests
      run: mvn clean verify

    - name: Generate test report
      uses: dorny/test-reporter@v1
      if: success() || failure()
      with:
        name: Maven Tests
        path: target/surefire-reports/*.xml
        reporter: java-junit

    - name: Upload coverage reports to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: target/site/jacoco/jacoco.xml

  security-scan:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: 'maven'

    - name: Run OWASP dependency check
      run: mvn org.owasp:dependency-check-maven:check

    - name: Run SpotBugs analysis
      run: mvn spotbugs:check

  deploy:
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        server-id: ossrh
        server-username: MAVEN_USERNAME
        server-password: MAVEN_PASSWORD

    - name: Deploy to Nexus
      run: mvn deploy -DskipTests
      env:
        MAVEN_USERNAME: ${{ secrets.MAVEN_USERNAME }}
        MAVEN_PASSWORD: ${{ secrets.MAVEN_PASSWORD }}
```

## Repository Management

### Nexus Repository Configuration

```xml
<repositories>
    <repository>
        <id>nexus-public</id>
        <url>https://nexus.example.com/repository/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>

<pluginRepositories>
    <pluginRepository>
        <id>nexus-public</id>
        <url>https://nexus.example.com/repository/public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>

<distributionManagement>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>https://nexus.example.com/repository/snapshots/</url>
    </snapshotRepository>
    <repository>
        <id>nexus-releases</id>
        <url>https://nexus.example.com/repository/releases/</url>
    </repository>
</distributionManagement>
```

## Best Practices

### 1. Dependency Management
- Use BOM imports for version management
- Avoid version conflicts with dependency management
- Exclude unnecessary transitive dependencies
- Use dependency convergence to ensure consistent versions

### 2. Build Optimization
- Enable parallel builds for faster compilation
- Use incremental builds when possible
- Optimize plugin configuration for your specific needs
- Cache dependencies and build artifacts

### 3. Quality Gates
- Implement enforce plugin rules
- Use code quality plugins (Checkstyle, PMD, SpotBugs)
- Set up test coverage thresholds
- Include security scanning in build pipeline

### 4. Release Management
- Use semantic versioning
- Implement release profiles
- Automate release process with Maven Release Plugin
- Sign artifacts for distribution

This comprehensive Maven guide provides enterprise-grade build management strategies, optimization techniques, and CI/CD integration patterns for modern Java development.