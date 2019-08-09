ARG OPENAM_VERSION=14.4.1
ARG OPENAM_ROOT=/usr/openam
ARG OPENAM_BASE=${OPENAM_ROOT}/config



# build OpenAM STS
# using alpine base image because maven image contains
# old git which has a recursive clone bug
#FROM maven:3.6.1-jdk-8 AS builder
FROM opensuse/tumbleweed:latest AS builder

ARG OPENAM_VERSION
ARG FORGEROCK_PARENT_VERSION=2.0.6

# check out OpenAM
WORKDIR /root
RUN zypper -q in -y git
RUN git clone --quiet --recursive https://github.com/OpenIdentityPlatform/OpenAM.git
WORKDIR /root/OpenAM
RUN git checkout -q ${OPENAM_VERSION}

# set up Maven
# FIXME
WORKDIR /root
RUN zypper -q in -y maven java-1_8_0-openjdk-devel tar gzip bzip2 nodejs

# build OpenAM
WORKDIR /root/OpenAM
RUN mvn -q -DskipTests -Dmaven.javadoc.skip=true -f OpenDJ/forgerock-parent install
RUN mvn -q -DskipTests -Dmaven.javadoc.skip=true -f OpenDJ install
RUN mvn -q -DskipTests -Dmaven.javadoc.skip=true -pl :openam-soap-sts-server -am install



# embed OpenAM STS into the official OpenAM core image
FROM openidentityplatform/openam:${OPENAM_VERSION}

ARG OPENAM_VERSION
ARG OPENAM_ROOT
ARG OPENAM_BASE

RUN echo ${OPENAM_VERSION}
USER root
RUN mkdir -p ${OPENAM_BASE}/soapstsdeployment
COPY --from=builder /root/OpenAM/openam-sts/openam-soap-sts/openam-soap-sts-server/target/openam-soap-sts-server-${OPENAM_VERSION}.war ${OPENAM_BASE}/soapstsdeployment

RUN chown -R ${OPENAM_USER} ${OPENAM_ROOT}
USER ${OPENAM_USER}
