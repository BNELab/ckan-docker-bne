FROM ghcr.io/bnelab/ckan-spatial-base:2.10.5-py3.10
LABEL maintainer="bnelab@bne.com"

# Set up environment variables
ENV APP_DIR=/srv/app
ENV TZ=UTC

# Set working directory
WORKDIR ${APP_DIR}

# requirements.txt files fixed until next releases
COPY req_fixes req_fixes

# CKAN configuration & extensions
## XLoader - 1.0.1-stable ##
## Harvest - v1.5.6 ##
## Geoview - v0.2.2 ##
## Spatial - v2.1.1 ##
## DCAT - v2.1.0 (Latest stable version of ckanext-dcat with minor fixes) ##
## Scheming - release-3.0.0 ##
## Resource dictionary - v1.0.2 (bnelab/Fixed version) ##
## Pages - v0.5.2 ##
## PDFView - 0.0.8 ##
## Fluent - v1.0.1  (bnelab/Forked stable version) ##
## OpenAPI - v1.0.0 (bnelab stable version) ##
## Scheming DCAT - v4.5.3 (bnelab/GeoDCAT-AP/NTI-RISP extended version) ##
## BNE - v1.3.1 (bnelab/API datos.abiertos) ##
RUN echo ${TZ} > /etc/timezone && \
    if ! [ /usr/share/zoneinfo/${TZ} -ef /etc/localtime ]; then cp /usr/share/zoneinfo/${TZ} /etc/localtime; fi && \
    # Install patch utility
    apt-get update && \
    apt-get install -y patch \
    # Install cron for scheduled tasks
    cron && \
    # Install CKAN extensions
    echo "ckan/ckanext-xloader" && \
    pip3 install --no-cache-dir -e git+https://github.com/bnelab/ckanext-xloader.git@1.0.1-stable#egg=ckanext-xloader && \ 
    pip3 install --no-cache-dir -r ${APP_DIR}/src/ckanext-xloader/requirements.txt && \
    pip3 install --no-cache-dir -U requests[security] && \
    echo "ckan/ckanext-harvest" && \
    pip3 install --no-cache-dir -e git+https://github.com/bnelab/ckanext-harvest.git@v1.5.6#egg=ckanext-harvest && \
    pip3 install --no-cache-dir -r ${APP_DIR}/src/ckanext-harvest/requirements.txt && \
    echo "ckan/ckanext-geoview" && \
    pip3 install --no-cache-dir -e git+https://github.com/ckan/ckanext-geoview.git@v0.2.2#egg=ckanext-geoview && \
    echo "ckan/ckanext-spatial" && \
    pip3 install --no-cache-dir -e git+https://github.com/ckan/ckanext-spatial.git@v2.3.0#egg=ckanext-spatial && \
    pip3 install --no-cache-dir -r ${APP_DIR}/src/ckanext-spatial/requirements.txt && \
    echo "ckan/ckanext-dcat (GeoDCAT-AP extended version)" && \
    pip3 install --no-cache-dir -e git+https://github.com/ckan/ckanext-dcat.git@v2.1.0#egg=ckanext-dcat && \
    pip3 install --no-cache-dir -r ${APP_DIR}/src/ckanext-dcat/requirements.txt && \
    echo "ckan/ckanext-scheming" && \
    pip3 install --no-cache-dir -e git+https://github.com/ckan/ckanext-scheming.git@release-3.0.0#egg=ckanext-scheming && \
    echo "bnelab/ckanext-resourcedictionary" && \
    pip3 install --no-cache-dir -e git+https://github.com/bnelab/ckanext-resourcedictionary.git@v1.0.2#egg=ckanext-resourcedictionary && \
    echo "ckan/ckanext-pages" && \
    pip3 install --no-cache-dir -e git+https://github.com/ckan/ckanext-pages.git@v0.5.2#egg=ckanext-pages && \
    echo "ckan/ckanext-pdfview" && \
    pip3 install --no-cache-dir -e git+https://github.com/ckan/ckanext-pdfview.git@0.0.8#egg=ckanext-pdfview && \
    echo "bnelab/ckanext-fluent" && \
    pip3 install --no-cache-dir -e git+https://github.com/bnelab/ckanext-fluent.git@v1.0.1#egg=ckanext-fluent && \
    echo "bnelab/ckanext-openapi" && \
    pip3 install --no-cache-dir -e git+https://github.com/bnelab/ckanext-openapi.git@v1.0.0#egg=ckanext-openapi && \
    echo "bnelab/ckanext-schemingdcat" && \
    pip3 install --no-cache-dir -e git+https://github.com/bnelab/ckanext-schemingdcat.git@v4.5.3#egg=ckanext_schemingdcat && \
    pip3 install --no-cache-dir -r ${APP_DIR}/src/ckanext-schemingdcat/requirements.txt && \
    echo "NaturalHistoryMuseum/ckanext-contact" && \
    pip3 install --no-cache-dir -e git+https://github.com/NaturalHistoryMuseum/ckanext-contact.git@v2.4.1#egg=ckanext-contact && \
    echo "bnelab/ckanext-bne" && \
    pip3 install --no-cache-dir -e git+https://github.com/bnelab/ckanext-bne.git@v1.3.1#egg=ckanext-bne && \
    # Remove system cache
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Used to configure the container environment by setting environment variables, creating users, running initialization scripts, .etc
COPY docker-entrypoint.d/* /docker-entrypoint.d/

# Apply any patches needed to CKAN core
COPY patches patches

RUN for d in $APP_DIR/patches/*; do \
    if [ -d $d ]; then \
        for f in `ls $d/*.patch | sort -g`; do \
            cd $SRC_DIR/`basename "$d"` && \
            if patch -R --dry-run -p1 < "$f"; then \
                echo "$0: Patch $f has already been applied or reversed, skipping..."; \
            else \
                echo "$0: Applying patch $f to $SRC_DIR/`basename $d`"; \
                sed -i 's/\r$//' "$f" && \
                patch -p1 < "$f" ; \
            fi \
        done ; \
    fi ; \
done

# Workers
## Update start_ckan.sh with custom workers
COPY setup/start_ckan.sh.override ${APP_DIR}/start_ckan.sh
RUN chmod +x ${APP_DIR}/start_ckan.sh

## Load workers supervisor configuration
COPY setup/workers/* /etc/supervisord.d/

# Start CKAN
CMD ["/bin/sh", "-c", "$APP_DIR/start_ckan.sh"]
