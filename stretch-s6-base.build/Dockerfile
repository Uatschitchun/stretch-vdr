###############################
# Target: stretch-s6-base     #
# Building _stretch-s6-base_  #
###############################

FROM debian:stretch-slim AS stretch-s6-base

ENV ARCH=amd64

ENV DEBIAN_FRONTEND noninteractive
ENV _clean="rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*"
ENV _apt_clean="eval apt-get clean && $_clean"

# Install needed
RUN apt-get update \
  && apt-get install --no-install-recommends -y \
        apt-utils \
        locales \
        tzdata \
        software-properties-common \
        dirmngr \
        apt-transport-https \
        gnupg \
  && $_apt_clean

# Set your local timezone
RUN echo "Europe/Berlin" > /etc/timezone && \
        dpkg-reconfigure -f noninteractive tzdata

# fix locale.
RUN sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

ENV LANG de_DE.UTF-8
ENV LANGUAGE de_DE:de
ENV LC_ALL de_DE.UTF-8

# Install s6-overlay
ADD https://github.com/just-containers/s6-overlay/releases/download/v1.21.4.0/s6-overlay-amd64.tar.gz /tmp/
RUN tar zxf /tmp/s6-overlay-amd64.tar.gz -C / && $_clean

ENTRYPOINT [ "/init" ]


################################################################################################################
# Target: builder											       #
# Build _builder_ image from stretch-s6-base for building vdr-epg-daemon*.deb used by vdr-epgd & mariadb-epglv #
################################################################################################################

FROM stretch-s6-base AS builder
WORKDIR /tmp
ENV ARCH=amd64


RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8 \
 && add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.23media.de/mariadb/repo/10.3/debian stretch main' \
 && apt-get update -q \
 && apt-get install -y --no-install-recommends \
        build-essential \
        nano \
        less \
        unzip \
        git \
        wget \
# VDR-EPGD
        libarchive-dev \
        zlib1g-dev \
#        libcurl4-nss-dev \
        libxml2-dev \
        libxslt1-dev \
        libmariadb-dev libmariadb-dev-compat \
        libjpeg-dev \
        uuid-dev \
        libjansson-dev \
        libimlib2-dev \
        libpython-dev python-dev \
        libssl-dev \
        libmicrohttpd-dev \
	debhelper \
	libcurl4-openssl-dev \
	dh-systemd \
	libsystemd-dev \
	dh-exec


#########################################################
# Target: build-epgd				        #
# Using _builder_ image to build vdr-epg-daemon's debs  #
# Is beeing used as delivery container for debs & logos #
#########################################################

FROM builder AS build-epgd
WORKDIR /tmp
ENV ARCH=amd64

RUN mkdir -p vdr-epg-daemon/ \
 && sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

# Copy debian source package from host
COPY deb-source/* vdr-epg-daemon/

# Build debs
RUN cd vdr-epg* \
 && dpkg-source -x vdr-*.dsc \
 && cd vdr-* \
 && dpkg-buildpackage -uc -us -j`nproc`

# Get channel logos
RUN cd /tmp \
 && wget https://github.com/FrodoVDR/channellogos/archive/master.tar.gz \
 && tar xzf master.tar.gz \
 && mkdir -p /var/epgd/www/channellogos \
 && cp -r channellogos-master/logos-orig/* /var/epgd/www/channellogos/


#######################
# Target: vdr-epgd    #
# Building _vdr-epgd_ #
#######################

FROM stretch-s6-base AS vdr-epgd

WORKDIR /tmp
ENV ARCH=amd64

RUN sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8 \
 && add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.23media.de/mariadb/repo/10.3/debian stretch main' \
 && apt-get update -q \
# && apt-get install -y --no-install-recommends wget \
# && cd /tmp \
# && wget https://github.com/FrodoVDR/channellogos/archive/master.tar.gz \
# && apt-get remove wget -y \
# && tar xzf master.tar.gz \
 && mkdir -p /var/epgd/www/channellogos
# Dieser Teil kann in den Builder und von dort die Logos ziehen
# && cp -r channellogos-master/logos-orig/* /var/epgd/www/channellogos/
COPY --from=build-epgd /var/epgd/www/channellogos/* /var/epgd/www/channellogos/

# !!!!Do NOT install all debs!!!!
COPY --from=build-epgd /tmp/vdr-epg-daemon/epgd_*.deb /tmp/
COPY --from=build-epgd /tmp/vdr-epg-daemon/epghttpd_*.deb /tmp/
RUN apt install ./*.deb -y --no-install-recommends \
 && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

VOLUME /epgd/config

COPY configs/vdr-epgd/configs/* /epgd/config/

# install runit scripts
COPY configs/vdr-epgd/epgd.sh /etc/services.d/epgd/run
COPY configs/vdr-epgd/epghttpd.sh /etc/services.d/epghttpd/run

EXPOSE 8888 9999

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
COPY --from=build-epgd /tmp/vdr-epg-daemon/mysql-plugin-epglv*.deb /tmp/
RUN apt install ./*.deb -y --no-install-recommends \
 && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV LANG de_DE.UTF-8
ENV LANGUAGE de_DE:de
ENV LC_ALL de_DE.UTF-8

COPY configs/mariadb-epglv/mysqlepgd.cnf /etc/mysql/conf.d/
COPY configs/mariadb-epglv/mysql-first-time.sql /docker-entrypoint-initdb.d/

VOLUME /var/lib/mysql

ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306
CMD ["mysqld"]

###########################
# Target: vdr-headless    #
# Building _vdr-headless_ #
###########################

FROM stretch-s6-base AS vdr-headless

WORKDIR /tmp
ENV ARCH=amd64

RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8 \
 && add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.23media.de/mariadb/repo/10.3/debian stretch main'

RUN echo deb https://packages.e-tobi.net/vdr-experimental stretch base vdr-multipatch addons > /etc/apt/sources.list.d/e-tobi.list \
 && echo deb-src https://packages.e-tobi.net/vdr-experimental stretch base vdr-multipatch addons >> /etc/apt/sources.list.d/e-tobi.list \
 && apt-key adv --recv-keys --keyserver keyring.debian.org --recv-keys 041390BC

RUN echo Package: * > /etc/apt/preferences.d/etobi \
 && echo Pin: release o=e-tobi.net >> /etc/apt/preferences.d/etobi \
 && echo Pin-Priority: 1000 >> /etc/apt/preferences.d/etobi

RUN sed -i '/^#.* de_DE.* /s/^#//' /etc/locale.gen && locale-gen

RUN apt-get update && \
        apt-get install --no-install-recommends -y \
        wget \
        vdr \
        rsyslog \
        vdr-plugin-epgsearch \
        vdr-plugin-femon \
        vdr-plugin-satip \
        vdr-plugin-live \
#       vdr-plugin-restfulapi \
        vdr-plugin-vnsiserver \
        vdr-plugin-vdrmanager \
        vdr-plugin-epg2vdr \
        vdr-plugin-svdrpservice \
        vdr-plugin-streamdev-server \
        vdr-plugin-svdrposd \
        libmariadbclient18 uuid \
        nano && \
        apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY vdr.sh /etc/services.d/vdr/run
COPY vdr-finish.sh /etc/services.d/vdr/finish
COPY rsyslog.sh /etc/services.d/rsyslog/run

# Configure the vdr user account and it's folders
#RUN groupmod -o -g 666 vdr \
# && usermod -o -u 666 vdr
# && install -o vdr -g vdr -d /recordings /vdr/config /vdr/cache

# add configs
COPY var/lib/vdr/* /var/lib/vdr/
COPY var/lib/vdr/plugins/streamdev-server/* /var/lib/vdr/plugins/streamdev-server/
COPY var/lib/vdr/plugins/vnsiserver/* /var/lib/vdr/plugins/vnsiserver/
COPY etc/vdr/conf.d/* /etc/vdr/conf.d/
COPY etc/vdr/conf.avail/* /etc/vdr/conf.avail/
COPY etc/rsyslog.conf /etc/

# 2004=???, 3000=streamdev, 6419=svdrp, 8002=restfulapi, 8008=live, 4010-4020/udp=satip, 34890=vnsiserver
#EXPOSE 2004 3000 6419 8002 8008 4010-4020/udp 34890
EXPOSE 2004 3000 6419 8008 4010-4020/udp 34890
VOLUME /var/lib/video/
#/etc/vdr /vdr/config /vdr/cache

ENTRYPOINT ["/init"]
