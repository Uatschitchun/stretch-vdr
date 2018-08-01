FROM uatschitchun/stretch-s6-base

WORKDIR /tmp

RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8 \
 && add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirror.23media.de/mariadb/repo/10.3/debian stretch main'

RUN echo deb https://packages.e-tobi.net/vdr-experimental stretch base vdr-multipatch addons > /etc/apt/sources.list.d/e-tobi.list \
 && echo deb-src https://packages.e-tobi.net/vdr-experimental stretch base vdr-multipatch addons >> /etc/apt/sources.list.d/e-tobi.list \
 && apt-key adv --recv-keys --keyserver keyring.debian.org --recv-keys 041390BC

RUN echo Package: * > /etc/apt/preferences.d/etobi \
 && echo Pin: release o=e-tobi.net >> /etc/apt/preferences.d/etobi \
 && echo Pin-Priority: 1000 >> /etc/apt/preferences.d/etobi

RUN apt-get update && \
	apt-get install --no-install-recommends -y \
	wget \
	vdr \
	rsyslog \
	vdr-plugin-epgsearch \
	vdr-plugin-femon \
	vdr-plugin-satip \
	vdr-plugin-live \
#	vdr-plugin-restfulapi \
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
