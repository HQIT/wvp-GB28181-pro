FROM dockerpull.pw/maven:3.8-openjdk-8 AS builder

WORKDIR /build
# 配置Maven使用阿里云镜像
RUN mkdir -p /root/.m2 && \
    echo '<?xml version="1.0" encoding="UTF-8"?>\
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" \
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" \
              xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">\
        <mirrors>\
            <mirror>\
                <id>aliyunmaven</id>\
                <mirrorOf>*</mirrorOf>\
                <name>阿里云公共仓库</name>\
                <url>https://maven.aliyun.com/repository/public</url>\
            </mirror>\
        </mirrors>\
    </settings>' > /root/.m2/settings.xml

# 先只复制pom文件并下载依赖
COPY gb28181-client/pom.xml .
RUN mvn dependency:go-offline

# 再复制源代码并编译
COPY gb28181-client/src ./src
RUN mvn clean package -DskipTests

FROM dockerpull.pw/openjdk:8-jdk-slim

WORKDIR /app
COPY --from=builder /build/target/gb28181-client-1.0-SNAPSHOT.jar /app/app.jar
RUN mkdir -p /app/extracted && \
    cd /app/extracted && \
    jar xf /app/app.jar && \
    rm /app/app.jar && \
    mkdir -p /app/config && \
    cp /app/extracted/BOOT-INF/classes/application-dev.yml /app/config/
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list && \
    apt-get update && apt-get install -y ffmpeg && \
    rm -rf /var/lib/apt/lists/*
ENV SPRING_CONFIG_LOCATION=/app/config/application-dev.yml

ENTRYPOINT ["java", "-cp", "/app/extracted/BOOT-INF/classes:/app/extracted/BOOT-INF/lib/*", "com.wydpp.GB28181ClientApplication"] 