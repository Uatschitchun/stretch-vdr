#######################
# Target: vdr-epgd    #
# Building _vdr-epgd_ #
#######################

FROM uatschitchun/stretch-s6-base AS vdr-epgd

WORKDIR /tmp
ENV ARCH=amd64

RUN sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8 \
 && add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.23media.de/mariadb/repo/10.3/debian stretch main' \
 && apt-get update -q \
 && mkdir -p /var/epgd/www/channellogos
COPY --from=uatschitchun/build-epgd /var/epgd/www/channellogos/* /var/epgd/www/channellogos/

# !!!!Do NOT install all debs!!!!
COPY --from=uatschitchun/build-epgd /tmp/vdr-epg-daemon/epgd_*.deb /tmp/
COPY --from=uatschitchun/build-epgd /tmp/vdr-epg-daemon/epghttpd_*.deb /tmp/
RUN apt install ./*.deb -y --no-install-recommends \
 && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

VOLUME /epgd/config

COPY configs/* /epgd/config/

# install runit scripts
COPY epgd.sh /etc/services.d/epgd/run
COPY epghttpd.sh /etc/services.d/epghttpd/run

EXPOSE 8888 9999
