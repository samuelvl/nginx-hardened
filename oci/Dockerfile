FROM docker.io/centos:centos8.1.1911

# Configuration variables
ENV NGINX_RPM_PACKAGE="nginx-1:1.14.1-9.module_el8.0.0+184+e34fea82" \
    NGINX_HTTP_PORT="8080" \
    NGINX_HTTPS_PORT="8443" \
    NGINX_CONF_DIR="/etc/nginx" \
    NGINX_STATIC_DIR="/usr/share/nginx/html" \
    NGINX_TLS_CERTS_DIR="/etc/pki/tls/certs" \
    NGINX_TLS_KEYS_DIR="/etc/pki/tls/private"

# Install nginx
RUN dnf install -y ${NGINX_RPM_PACKAGE} &&\
    rm -r /var/cache/dnf

# Create a non-root user
RUN useradd --uid 1001 --system --create-home \
    --user-group --groups tty --shell /sbin/nologin noroot

# Redirect logs to stdout and stderr
RUN ln -sf /dev/stdout /var/log/nginx/access.log &&\
    ln -sf /dev/stderr /var/log/nginx/error.log

# Update nginx configuration
COPY --chown=root:root \
    configuration ${NGINX_CONF_DIR}

RUN find ${NGINX_CONF_DIR} -type d -exec chmod 0755 {} \; &&\
    find ${NGINX_CONF_DIR} -type f -exec chmod 0644 {} \;

# Upload static website content
COPY --chown=root:root \
    website ${NGINX_STATIC_DIR}

RUN find ${NGINX_STATIC_DIR} -type d -exec chmod 0755 {} \; &&\
    find ${NGINX_STATIC_DIR} -type f -exec chmod 0644 {} \;

# Create directory structure for certificates
COPY --chown=root:root \
    certs/template/*.pem ${NGINX_TLS_CERTS_DIR}

COPY --chown=root:root \
    certs/template/*.crt ${NGINX_TLS_CERTS_DIR}

COPY --chown=root:root \
    certs/template/*.key ${NGINX_TLS_KEYS_DIR}

# Switch to nginx user
USER nginx

# Expose nginx ports
EXPOSE ${NGINX_HTTP_PORT} ${NGINX_HTTPS_PORT}

# Run nginx at startup
CMD ["nginx", "-g", "daemon off;"]