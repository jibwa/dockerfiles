FROM adoptopenjdk:11.0.9_11-jdk-hotspot-focal@sha256:22fda5482af1e2364248517a9d22991cecd640965938828534e5233b8d9b13eb
ARG VERSION=2.0.95
USER root
RUN apt-get update && apt-get install -y --no-install-recommends git ca-certificates
RUN git clone -b ${VERSION} https://github.com/linkedin/cruise-control.git
RUN cd cruise-control && ./gradlew jar copyDependantLibs

RUN mv -v /cruise-control/cruise-control/build/libs/cruise-control-*.jar \
  /cruise-control/cruise-control/build/libs/cruise-control.jar
RUN mv -v /cruise-control/cruise-control/build/dependant-libs/cruise-control-metrics-reporter-*.jar \
  /cruise-control/cruise-control/build/dependant-libs/cruise-control-metrics-reporter.jar

FROM node:10.16
RUN mkdir /src && cd /src && git clone https://github.com/linkedin/cruise-control-ui.git
WORKDIR /src/cruise-control-ui
RUN git fetch origin
RUN git checkout master
RUN git pull
RUN git rev-parse HEAD
RUN npm install
RUN npm run build

# The container is made to work with github.com/Yolean/kubernetes-kafka, so we try to use a common base
FROM adoptopenjdk:11.0.9_11-jdk-hotspot-focal@sha256:22fda5482af1e2364248517a9d22991cecd640965938828534e5233b8d9b13eb
ARG SOURCE_REF
ARG SOURCE_TYPE
ARG DOCKERFILE_PATH
ARG VERSION

RUN mkdir -p /opt/cruise-control /opt/cruise-control/cruise-control-ui
COPY --from=0 /cruise-control/cruise-control/build/libs/cruise-control.jar /opt/cruise-control/cruise-control/build/libs/cruise-control.jar
COPY --from=0 /cruise-control/config /opt/cruise-control/config
COPY --from=0 /cruise-control/kafka-cruise-control-start.sh /opt/cruise-control/
COPY --from=0 /cruise-control/cruise-control/build/dependant-libs /opt/cruise-control/cruise-control/build/dependant-libs
COPY opt/cruise-control /opt/cruise-control/
COPY --from=1 /src/cruise-control-ui/dist /opt/cruise-control/cruise-control-ui/dist
RUN echo "local,localhost,/kafkacruisecontrol" > /opt/cruise-control/cruise-control-ui/dist/static/config.csv

EXPOSE 8090
CMD [ "/opt/cruise-control/start.sh" ]
