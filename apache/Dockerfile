FROM owasp/modsecurity:2.9.6-202209170109 as release

ARG RELEASE

# hadolint ignore=DL3008,SC2016
RUN set -eux; \
    apt-get update; \
    apt-get -y install --no-install-recommends \
      ca-certificates \
      curl \
      gnupg \
      iproute2; \
    update-ca-certificates -f; \
    mkdir /opt/owasp-crs; \
    curl -SL -q https://github.com/coreruleset/coreruleset/archive/v${RELEASE}.tar.gz -o v${RELEASE}.tar.gz; \
    curl -SL -q https://github.com/coreruleset/coreruleset/releases/download/v${RELEASE}/coreruleset-${RELEASE}.tar.gz.asc -o coreruleset-${RELEASE}.tar.gz.asc; \
    gpg --fetch-key https://coreruleset.org/security.asc; \
    gpg --verify coreruleset-${RELEASE}.tar.gz.asc v${RELEASE}.tar.gz; \
    tar -zxf v${RELEASE}.tar.gz --strip-components=1 -C /opt/owasp-crs; \
    rm -f v${RELEASE}.tar.gz coreruleset-${RELEASE}.tar.gz.asc; \
    mv -v /opt/owasp-crs/crs-setup.conf.example /opt/owasp-crs/crs-setup.conf; \
    ln -sv /opt/owasp-crs /etc/modsecurity.d/; \
    sed -i -E 's/(Listen) [0-9]+/\1 ${PORT}/g' /usr/local/apache2/conf/httpd.conf; \
    rm -rf /var/lib/apt/lists/*

FROM owasp/modsecurity:2.9.6-202209170109

LABEL maintainer="Felipe Zipitria <felipe.zipitria@owasp.org>"


# overridden variables
ENV ERRORLOG='/proc/self/fd/2' \
    USER=www-data \
    GROUP=www-data \
    MODSEC_PCRE_MATCH_LIMIT=100000 \
    MODSEC_PCRE_MATCH_LIMIT_RECURSION=100000

# CRS specific variables
ENV PARANOIA=1 \
    ANOMALY_INBOUND=5 \
    ANOMALY_OUTBOUND=4 \
    BLOCKING_PARANOIA=1

COPY src/etc/modsecurity.d/*.conf /etc/modsecurity.d/
COPY src/opt/modsecurity/activate-rules.sh /opt/modsecurity/
COPY apache/conf/extra/*.conf /usr/local/apache2/conf/extra/
COPY apache/docker-entrypoint.sh /
COPY --from=release /opt/owasp-crs /opt/owasp-crs

RUN set -eux; ln -s /opt/owasp-crs /etc/modsecurity.d/

ENTRYPOINT ["/docker-entrypoint.sh"]

# Use httpd-foreground from upstream
CMD ["httpd-foreground"]
