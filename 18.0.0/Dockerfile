FROM quay.io/keycloak/keycloak:18.0.0 as builder

ENV KC_HEALTH_ENABLED=true
ENV KC_FEATURES=openshift_integration
ENV KC_DB=postgres
ENV KC_CACHE_CONFIG_FILE=cache-ispn-jdbc-ping.xml

COPY --chown=1000:0 ../bcit-keycloak-theme.jar /opt/keycloak/providers/bcit-keycloak-theme.jar
COPY --chown=1000:0 ../cache-ispn-jdbc-ping.xml /opt/keycloak/conf/cache-ispn-jdbc-ping.xml

RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:18.0.0

USER root
RUN microdnf update -y && microdnf install -y tar gzip wget && microdnf clean all

LABEL maintainer="jesse@weisner.ca, chriswood.ca@gmail.com"
LABEL build_id="1654206659"

# Add docker-entrypoint script base
RUN mkdir /docker-entrypoint.d/ \
 && wget -qO- https://github.com/itsbcit/docker-entrypoint/releases/download/v1.6/docker-entrypoint.tar.gz | tar xvz -C / \
 && chmod -R 555 /docker-entrypoint.* \
 && echo "UTC" > /etc/timezone \
 && chmod 664 /etc/passwd \
              /etc/group \
              /etc/shadow \
              /etc/timezone \
 && chown 0:0 /etc/shadow \
 && chmod 775 /etc

# Add Tini
ADD https://github.com/krallin/tini/releases/download/v0.19.0/tini-static-amd64 /tini
RUN chmod +x /tini \
 && ln -s /tini /sbin/tini 

USER keycloak 
COPY --from=builder /opt/keycloak/ /opt/keycloak/
WORKDIR /opt/keycloak

ENTRYPOINT ["/tini", "--", "/docker-entrypoint.sh"]
CMD ["/opt/keycloak/bin/kc.sh", "start"]

