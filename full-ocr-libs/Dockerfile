# Licensed under the Apache License, Version 2.0 (the "License"); you may not
# use this file except in compliance with the License. You may obtain a copy of
# the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations under
# the License.
FROM ubuntu:focal as base
RUN apt-get update

FROM base as dependencies

# "random" uid/gid hopefully not used anywhere else
ARG UID_GID="35002:35002"
ARG JRE='openjdk-17-jre-headless'

WORKDIR "/tmp/"

RUN DEBIAN_FRONTEND=noninteractive apt-get -y install $JRE maven gdal-bin tesseract-ocr \
        tesseract-ocr-eng tesseract-ocr-ita tesseract-ocr-fra tesseract-ocr-spa tesseract-ocr-deu

RUN echo ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true | debconf-set-selections \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y xfonts-utils fonts-freefont-ttf fonts-liberation ttf-mscorefonts-installer wget cabextract

FROM dependencies as fetch_tika
ARG TIKA_VERSION
ARG TIKA_PACKAGE_NAME=tika
ARG CHECK_SIG=true

RUN DEBIAN_FRONTEND=noninteractive apt-get -y install gnupg2 wget 

RUN wget "https://www.apache.org/dyn/closer.lua/tika/${TIKA_VERSION}/${TIKA_PACKAGE_NAME}-${TIKA_VERSION}-src.zip" -o "/tmp/${TIKA_PACKAGE_NAME}-${TIKA_VERSION}-src.zip"
RUN wget -t 10 --max-redirect 1 --retry-connrefused -qO- https://downloads.apache.org/tika/KEYS | gpg --import

RUN if [ "$CHECK_SIG" = "true" ] ; then gpg --verify ${TIKA_PACKAGE_NAME}-${TIKA_VERSION}-src.zip.asc ${TIKA_PACKAGE_NAME}-${TIKA_VERSION}-src.zip; fi

WORKDIR "/tmp/${TIKA_PACKAGE_NAME}-${TIKA_VERSION}/"
RUN mvn install
COPY "./tika-server/target/${TIKA_PACKAGE_NAME}-server-${TIKA_VERSION}.jar" "/${TIKA_PACKAGE_NAME}-${TIKA_VERSION}.jar"

FROM dependencies as runtime
RUN apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
ARG TIKA_VERSION
ENV TIKA_VERSION=$TIKA_VERSION
ARG TIKA_PACKAGE_NAME
ENV TIKA_PACKAGE_NAME=$TIKA_PACKAGE_NAME
USER $UID_GID
EXPOSE 9998
ENTRYPOINT [ "/bin/sh", "-c", "exec java -jar /${TIKA_PACKAGE_NAME}-${TIKA_VERSION}.jar -h 0.0.0.0 $0 $@"]

LABEL maintainer="Apache Tika Developers dev@tika.apache.org"

