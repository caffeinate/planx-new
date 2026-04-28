# XXX: pg_cron pinned to commit 465b38c (v1.6.7) - please update when bumping image
FROM docker.io/postgres:16-bullseye

ENV PG_MAJOR=16
ENV POSTGIS_MAJOR=3
ENV POSTGIS_VERSION=3.5.2+dfsg-1.pgdg110+1

RUN apt-get update \
      && apt-cache showpkg postgresql-$PG_MAJOR-postgis-$POSTGIS_MAJOR \
      && apt-get install -y --no-install-recommends \
           # ca-certificates: for accessing remote raster files;
           #   fix: https://github.com/postgis/docker-postgis/issues/307
           ca-certificates \
           postgresql-$PG_MAJOR-postgis-$POSTGIS_MAJOR=$POSTGIS_VERSION \
           postgresql-$PG_MAJOR-postgis-$POSTGIS_MAJOR-scripts

RUN mkdir -p /docker-entrypoint-initdb.d
COPY ./initdb-postgis.sh /docker-entrypoint-initdb.d/10_postgis.sh
COPY ./update-postgis.sh /usr/local/bin


# install build deps, build pg_cron and remove build deps to keep image small
RUN apt-get install -y --no-install-recommends build-essential git postgresql-server-dev-$PG_MAJOR \
  # as a safety catch, this build will automatically fail if we fall far behind in the pg_cron commit history
  && git clone https://github.com/citusdata/pg_cron.git --depth 50 /tmp/pg_cron \
  && cd /tmp/pg_cron \
  && git checkout 465b38c737f584d520229f5a1d69d1d44649e4e5 \
  && make \
  && make install \
  && rm -rf /tmp/pg_cron \
  && apt-get purge -y --auto-remove build-essential git \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*
