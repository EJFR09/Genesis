FROM eclipse-temurin:21-jre-alpine-3.23

  

ARG APP_DIR=/app

ARG JAR_FILE=build/libs/*.jar

ARG APP_PORT=8080

  

ENV TZ=America/Asuncion

ENV HOME=/app

ENV JDK_JAVA_OPTIONS="-XX:+UseContainerSupport -Dfile.encoding=UTF-8"

  

RUN apk add --no-cache tzdata \

    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \

    && echo $TZ > /etc/timezone \

    && adduser -S -D -H -G root usermame \

    && mkdir -p ${APP_DIR} \

    && chgrp -R 0 ${APP_DIR} \

    && chmod -R g=u ${APP_DIR}

  

WORKDIR ${APP_DIR}

  

COPY --chown=username:0 ${JAR_FILE} ${APP_DIR}/app.jar

  

EXPOSE ${APP_PORT}

  

USER username

  

ENTRYPOINT ["java", "-jar", "/app/app.jar"]