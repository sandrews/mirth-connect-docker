# https://hub.docker.com/r/brandonstevens/mirth-connect/
FROM openjdk:8

ARG MIRTH_LOGO=mirth-logo.png

ENV MIRTH_CONNECT_VERSION 3.7.1.b243

# Mirth Connect is run with user `connect`, uid = 1000
# If you bind mount a volume from the host or a data container,
# ensure you use the same uid
RUN useradd -u 1000 mirth

# update and install requirements
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates wget 

# grab gosu for easy step-down from root
RUN set -eux; \
	apt-get install -y gosu; \
	rm -rf /var/lib/apt/lists/*; \
# verify that the binary works
	gosu nobody true

VOLUME /opt/mirth-connect/appdata

RUN \
  cd /tmp && \
  wget http://downloads.mirthcorp.com/connect/$MIRTH_CONNECT_VERSION/mirthconnect-$MIRTH_CONNECT_VERSION-unix.tar.gz && \
  tar xvzf mirthconnect-$MIRTH_CONNECT_VERSION-unix.tar.gz && \
  rm -f mirthconnect-$MIRTH_CONNECT_VERSION-unix.tar.gz && \
  mv Mirth\ Connect/* /opt/mirth-connect/ && \
  chown -R mirth /opt/mirth-connect

RUN rm -f /opt/mirth-connect/public_html/images/mirthconnectlogowide.png
RUN rm -f /opt/mirth-connect/public_api_html/images/mirthconnectlogowide.png

COPY $MIRTH_LOGO /opt/mirth-connect/public_html/images/mirthconnectlogowide.png
COPY $MIRTH_LOGO  /opt/mirth-connect/public_api_html/images/mirthconnectlogowide.png

RUN \
	cd /opt/mirth-connect/webapps && \
	mkdir tmp && \
	mv webadmin.war ./tmp && \ 
	cd tmp && \
	jar -xvf webadmin.war && \
	rm -rf ./images/mirthconnectlogowide.png && \
	rm -rf ./css/mirthconnectlogowide.png && \
	cp /opt/mirth-connect/public_html/images/mirthconnectlogowide.png ./images/mirthconnectlogowide.png && \
	cp /opt/mirth-connect/public_html/images/mirthconnectlogowide.png ./css/mirthconnectlogowide.png && \
	rm -rf webadmin.war && \ 
	jar -cvf  webadmin.war * && \
	mv webadmin.war ../ && \
	cd .. && \
	rm -rf ./tmp 



WORKDIR /opt/mirth-connect

# set spool volume for messages exchange with Docker host ----
RUN mkdir /var/spool/mirth
RUN chown -R mirth /var/spool/mirth
VOLUME /var/spool/mirth
# ----

EXPOSE 8080 8443

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["java", "-jar", "mirth-server-launcher.jar"]

#RUN  for f in `find ./appdata/temp -iname mirthconnectlogowide.png`; do cp /opt/mirth-connect/public_html/images/mirthconnectlogowide.png $f;done
