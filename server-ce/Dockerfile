# ---------------------------------------------
# Overleaf Community Edition (overleaf/overleaf)
# ---------------------------------------------

ARG OVERLEAF_BASE_TAG=undefined443/sharelatex-base:latest
FROM $OVERLEAF_BASE_TAG

LABEL org.opencontainers.image.source https://github.com/undefined443/overleaf

WORKDIR /overleaf

# Add required source files
# -------------------------
ADD server-ce/genScript.js /overleaf/genScript.js
ADD server-ce/services.js /overleaf/services.js
ADD package.json package-lock.json /overleaf/
ADD libraries/ /overleaf/libraries/
ADD services/ /overleaf/services/

# Store the revision
# ------------------
ARG MONOREPO_REVISION
RUN echo "monorepo-server-ce,$MONOREPO_REVISION" > /var/www/revisions.txt

# Add npm patches
# -----------------------
ADD patches/ /overleaf/patches

# Install npm dependencies
# ------------------------
ENV CYPRESS_INSTALL_BINARY=0
RUN node genScript install | bash

# Compile
# --------------------
RUN node genScript compile | bash

# Copy runit service startup scripts to its location
# --------------------------------------------------
ADD server-ce/runit /etc/service

# Copy runit global settings to its location
# ------------------------------------------
ADD server-ce/config/env.sh /etc/overleaf/env.sh

# Configure nginx
# ---------------
ADD server-ce/nginx/nginx.conf.template /etc/nginx/templates/nginx.conf.template
ADD server-ce/nginx/overleaf.conf /etc/nginx/sites-enabled/overleaf.conf
ADD server-ce/nginx/clsi-nginx.conf /etc/nginx/sites-enabled/clsi-nginx.conf


# Configure log rotation
# ----------------------
ADD server-ce/logrotate/overleaf /etc/logrotate.d/overleaf
RUN chmod 644 /etc/logrotate.d/overleaf

# Configure cron tasks
# ----------------------
ADD server-ce/cron /overleaf/cron
ADD server-ce/config/crontab-history /etc/cron.d/crontab-history
RUN chmod 600 /etc/cron.d/crontab-history
ADD server-ce/config/crontab-deletion /etc/cron.d/crontab-deletion
RUN chmod 600 /etc/cron.d/crontab-deletion

# Copy Phusion Image startup and shutdown scripts to their locations
# ------------------------------------------------------------------
COPY server-ce/init_scripts/ /etc/my_init.d/
COPY server-ce/init_preshutdown_scripts/ /etc/my_init.pre_shutdown.d/

# Copy app settings files
# -----------------------
COPY server-ce/config/settings.js /etc/overleaf/settings.js

# Copy history-v1 files
# -----------------------
COPY server-ce/config/production.json /overleaf/services/history-v1/config/production.json
COPY server-ce/config/custom-environment-variables.json /overleaf/services/history-v1/config/custom-environment-variables.json

# Copy grunt thin wrapper
# -----------------------
ADD server-ce/bin/grunt /usr/local/bin/grunt
RUN chmod +x /usr/local/bin/grunt

# Copy history helper scripts
# ---------------------------
ADD server-ce/bin/flush-history-queues /overleaf/bin/flush-history-queues
RUN chmod +x /overleaf/bin/flush-history-queues
ADD server-ce/bin/force-history-resyncs /overleaf/bin/force-history-resyncs
RUN chmod +x /overleaf/bin/force-history-resyncs

# Copy font files
# ---------------
ADD server-ce/fonts/ /usr/local/share/fonts/
RUN ln -s /usr/local/texlive/2024/texmf-var/fonts/conf/texlive-fontconfig.conf /etc/fonts/conf.d/09-texlive.conf && \
    fc-cache -fv && \
    apt-get update && \
    apt-get install -y inkscape python3-pygments && \
    echo "shell_escape = t" >> /usr/local/texlive/2024/texmf.cnf

# File that controls open|closed status of the site
# -------------------------------------------------
ENV SITE_MAINTENANCE_FILE "/etc/overleaf/site_status"
RUN touch $SITE_MAINTENANCE_FILE

# Set Environment Variables
# --------------------------------
ENV OVERLEAF_CONFIG /etc/overleaf/settings.js

ENV WEB_API_USER "overleaf"
ENV ADMIN_PRIVILEGE_AVAILABLE "true"

ENV OVERLEAF_APP_NAME "Overleaf Community Edition"

ENV OPTIMISE_PDF "true"

# Phusion Image timeouts before sending SIGKILL to processes
# ----------------------------------------------------------
ENV KILL_PROCESS_TIMEOUT 55
ENV KILL_ALL_PROCESSES_TIMEOUT 55

ENV NODE_ENV "production"
ENV LOG_LEVEL "info"


EXPOSE 80

ENTRYPOINT ["/sbin/my_init"]
