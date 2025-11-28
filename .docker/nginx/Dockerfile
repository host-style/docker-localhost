FROM nginx:latest

# Install OpenSSL for generating SSL certificates
RUN apt-get update && apt-get install -y openssl && rm -rf /var/lib/apt/lists/*

# Get user/group IDs from build args
ARG PUID=1000
ARG PGID=1000

# Create a user matching host permissions
RUN groupmod -o -g ${PGID} nginx || groupadd -g ${PGID} nginx \
    && usermod -o -u ${PUID} -g nginx nginx || useradd -u ${PUID} -g nginx nginx

# Declare volumes
VOLUME ["/etc/nginx/certs", "/var/log/nginx"]

# Create entrypoint script
RUN echo '#!/bin/bash\n\
if [ ! -f /etc/nginx/certs/localhost.crt ]; then\n\
  echo "Generating SSL certificates..."\n\
  mkdir -p /etc/nginx/certs\n\
  openssl req -x509 -nodes -days 365 -newkey rsa:2048 \\\n\
    -keyout /etc/nginx/certs/localhost.key \\\n\
    -out /etc/nginx/certs/localhost.crt \\\n\
    -subj "/C=BR/ST=State/L=City/O=Organization/CN=localhost"\n\
  chmod 644 /etc/nginx/certs/localhost.crt\n\
  chmod 600 /etc/nginx/certs/localhost.key\n\
  echo "SSL certificates generated successfully"\n\
else\n\
  echo "SSL certificates already exist"\n\
fi\n\
\n\
# Ensure log directory has correct permissions\n\
mkdir -p /var/log/nginx\n\
chown -R nginx:nginx /var/log/nginx\n\
chmod -R 755 /var/log/nginx\n\
\n\
exec nginx -g "daemon off;"\n\
' > /docker-entrypoint.sh && chmod +x /docker-entrypoint.sh

# Expose ports
EXPOSE 80 443

ENTRYPOINT ["/docker-entrypoint.sh"]
