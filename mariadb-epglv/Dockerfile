############################
# Target: mariadb-epglv    #
# Building _mariadb-epglv_ #
############################

FROM mariadb:10.3 AS mariadb-epglv

WORKDIR /tmp
ENV ARCH=amd64

RUN apt-get update -q \
 && apt-get install -y --no-install-recommends locales

RUN echo "Europe/Berlin" > /etc/timezone \
 && dpkg-reconfigure -f noninteractive tzdata \
 && sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

# !!!!copy over epglv from build-epgd!!!!
COPY --from=uatschitchun/build-epgd /tmp/vdr-epg-daemon/mysql-plugin-epglv*.deb /tmp/
RUN apt install ./*.deb -y --no-install-recommends \
 && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV LANG de_DE.UTF-8
ENV LANGUAGE de_DE:de
ENV LC_ALL de_DE.UTF-8

COPY configs/mysqlepgd.cnf /etc/mysql/conf.d/
COPY configs//mysql-first-time.sql /docker-entrypoint-initdb.d/

VOLUME /var/lib/mysql

ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306
CMD ["mysqld"]
