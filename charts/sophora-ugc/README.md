# Sophora UGC

This chart deploys Sophora UGC

## What you need (requirements)

This chart requires the following already present in the target namespace:

* An ImagePullSecret for the subshell Docker Registry
* A secret containing username and password for the sophora server and a username and password for the database.

## Example values.yaml

```yaml
image:
  repository: docker.subshell.com/ugc/ugc
  pullPolicy: Always
  tag: "3.1.1"

service:
  jolokia:
    clusterIP: None
  webapp:
    type: LoadBalancer

sophora:
  authentication:
    secret:
      name: secret-name
      usernameKey: username
      passwordKey: password
      database:
        usernameKey: database-user
        passwordKey: database-password
  ugc:
    logback: |
      <?xml version="1.0" encoding="UTF-8"?>
      <configuration scan="true">
          <jmxConfigurator />

          <!-- Propagate level settings to java.util.logging. -->
          <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator"/>

          <appender name="logfile" class="ch.qos.logback.core.rolling.RollingFileAppender">
              <File>/logs/ugc-webapp.log</File>
              <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                  <fileNamePattern>/logs/ugc.%d{yyyy-MM-dd}.log</fileNamePattern>
                  <maxHistory>7</maxHistory>
              </rollingPolicy>
              <encoder>
                  <pattern>[%d{"yyyy-MM-dd HH:mm:ss.SSSX",UTC}, %-5p] [%t] [%X{ID}] %.30c:%L: %m%n</pattern>
              </encoder>
          </appender>
          <appender name="errorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
              <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
                  <level>ERROR</level>
              </filter>
              <File>/logs/error.log</File>
              <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                  <fileNamePattern>/logs/error.%d{yyyy-MM-dd}.log</fileNamePattern>
                  <maxHistory>7</maxHistory>
              </rollingPolicy>
              <encoder>
                  <pattern>[%d{"yyyy-MM-dd HH:mm:ss.SSSX",UTC}, %-5p] [%t] [%X{ID}] %.40c:%L: %m%n</pattern>
              </encoder>
          </appender>

          <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
              <encoder>
                  <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level [%X{ID}] %logger{36} - %msg%n</pattern>
              </encoder>
          </appender>

          <logger name="com.subshell.sophora.ugc" level="INFO" />

          <root level="WARN">
              <appender-ref ref="STDOUT" />
              <appender-ref ref="logfile" />
              <appender-ref ref="errorLog" />
          </root>
      </configuration>

    config:
      sophora-server:
        host: "http://localhost:1196"

      # configuration of the database connection. Adapt this to match with your settings
      database :
        # url of the jdbc connection, the last part is the name of the database 
        url : "jdbc:mysql://localhost:3306/usercontent"

      # configure the port of the embedded jetty
      server:
        port: 9080

      # ratings, image uploads and comments can be enabled for a list of node types
      rating:
        primaryTypes: ["sophora-content-nt:story"]

resources:
  requests:
    memory: "3.5G"
    cpu: "0.5"
  limits:
    memory: "3.5G"
```
