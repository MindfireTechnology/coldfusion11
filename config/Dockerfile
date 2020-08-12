# Ubuntu 18.04 LTS-based image that installs OpenJDK 8, libstdc++5, Coldfusion 11, Hotfix 15, and Commandbox
FROM fedora/apache

# ACF runs on 8500; ACF 11 PDFG runs on 8987; other versions use other ports (8988?)
EXPOSE 8500

LABEL version="@version@"
LABEL maintainer "Mindfire Technology <Info@MindfireTechnology.com>"
LABEL repository "https://github.com/MindfireTechnology/coldfusion11"

# Commandbox install folder
ENV BIN_DIR=/usr/bin
# Build folder
ENV TEMP_BUILD_DIR /root/tmp-build/
# OpenJDK Install path
ENV JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk/

WORKDIR $TEMP_BUILD_DIR

ADD https://github.com/capnregex/cold_fusion_installers/releases/download/CF11.15.01.13/ColdFusion_11_WWEJ_linux64_011315.bin $TEMP_BUILD_DIR

RUN yum -y update && \
	yum -y install tar tzdata nano unzip curl libstdc++.so.5 java-1.8.0-openjdk && \
	rm -rf /var/cache/oracle-jdk8-installer;

COPY ./build/silent.properties $TEMP_BUILD_DIR

# Run ACF11 installer and delete the installer
RUN  chmod +x ./ColdFusion_11_WWEJ_linux64_011315.bin && \
	./ColdFusion_11_WWEJ_linux64_011315.bin LAX_VM ${JAVA_HOME}/bin/java -f silent.properties && \
	rm -rf $TEMP_BUILD_DIR

# Download ACF Hotfix, copy hotfix.properties and ACF11 runtime files

ADD https://cfdownload.adobe.com/pub/adobe/coldfusion/11/hotfix_015.jar /opt/coldfusion11/jre/bin/
COPY ./hotfix/ /opt/coldfusion11/jre/bin/
COPY ./build/neo-runtime.xml /opt/coldfusion11/cfusion/lib/
COPY ./build/jvm.config /opt/coldfusion11/cfusion/bin/
COPY cf-docker.sh /opt/coldfusion11/

# Install ACF11 Hotfix and then delete it and the ACF11 bundled JRE
RUN java -jar /opt/coldfusion11/jre/bin/hotfix_015.jar -i silent -f /opt/coldfusion11/jre/bin/hotfix.properties && \
	rm -rf /opt/coldfusion11/jre/* && \
	chmod +x /opt/coldfusion11/cf-docker.sh

# Install mod_cfml and server.xml default to use it and move the docRoot to /var/www
ADD https://github.com/viviotech/mod_cfml/blob/master/java/mod_cfml-valve_v1.1.05.jar?raw=true /opt/coldfusion11/cfusion/runtime/lib
COPY ./build/server.xml /opt/coldfusion11/cfusion/runtime/conf/

# Install Commandbox and CFConfig

RUN	curl --location 'https://www.ortussolutions.com/parent/download/commandbox/type/bin' -o /tmp/box.zip && \
	unzip -o /tmp/box.zip -d ${BIN_DIR} && chmod +x ${BIN_DIR}/box && \
	box install commandbox-cfconfig && \
    rm /tmp/box.zip && \
    echo "$(box version) successfully installed"

WORKDIR /opt/coldfusion11

CMD ["/opt/coldfusion11/cf-docker.sh"]
