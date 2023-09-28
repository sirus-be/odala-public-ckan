###################
### Extensions ####
###################
FROM ghcr.io/keitaroinc/ckan:2.9.5 as extbuild

WORKDIR ${APP_DIR}

ENV CKAN__PLUGINS envvars image_view text_view recline_view datastore datapusher openapi_view dcat

# Switch to the root user
USER root

# Install any system packages necessary to build extensions
# Make sure we install python 3.8, cause CKAN is not compatible with 3.9
RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/v3.13/main \
        python3-dev=3.8.10-r0 

############
### MAIN ###
############
FROM ghcr.io/keitaroinc/ckan:2.9.5

# Switch to the root user
USER root

ADD ckanext/ckanext-openapi /src/ckanext/ckanext-openapi
ADD licenses/licenses.json licenses.json
# add extensions https://github.com/keitaroinc/docker-ckan#extending-ckan-docker-images

COPY ["start_ckan.sh", "/srv/app/start_ckan.sh"]

# Instal DCAT extension https://github.com/ckan/ckanext-dcat.git
WORKDIR ${APP_DIR}/src/ckanext/ckanext-dcat
RUN pip install -e git+https://github.com/ckan/ckanext-dcat.git#egg=ckanext-dcat
WORKDIR ${APP_DIR}/src/ckanext-dcat
RUN pip install -r requirements.txt

# Modify production.ini
WORKDIR ${APP_DIR}
RUN echo "$PWD"
RUN ckan config-tool ${APP_DIR}/production.ini "ckan.auth.create_user_via_web = false" && \
        ckan config-tool ${APP_DIR}/production.ini "licenses_group_url = file://licenses.json" && \
        ckan config-tool ${APP_DIR}/production.ini "ckan.plugins = envvars image_view text_view recline_view datastore datapusher dcat" && \
        chown -R ckan:ckan /srv/app

# Switch to the ckan user
USER ckan

CMD ["/srv/app/start_ckan.sh"]