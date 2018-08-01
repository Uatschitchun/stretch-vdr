FROM mariadb:10.3

WORKDIR /tmp

RUN mkdir -p epglv/src
COPY epglv/* epglv/
COPY epglv/src/* epglv/src/

RUN apt-get update -q &&\
    apt-get install -y apt-utils &&\
    apt-get install -y locales &&\
#    echo "Europe/Berlin" > /etc/timezone &&\
#    dpkg-reconfigure -f noninteractive tzdata &&\
#    echo "de_DE.UTF-8 UTF-8" >> /etc/locale.gen; locale-gen &&\
#    \
    apt-get install -y build-essential \
            mysql-common libmariadbclient-dev-compat libmariadbclient18 zlib1g zlib1g-dev libcrypto++-dev libssl-dev

#RUN    cd /tmp/epglv && \
#            make all && \
#            make install &&\
#      \
#    apt-get remove -y \
#            build-essential \
#                libcrypto++-dev \
#                libssl-dev \
#            libmariadbclient-dev zlib1g-dev &&\
#      \
#    apt-get autoremove -y && apt-get clean && apt-get purge && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


#ENV LANG de_DE.UTF-8
#ENV LANGUAGE de_DE:de
#ENV LC_ALL de_DE.UTF-8

COPY mysqlepgd.cnf /etc/mysql/conf.d/
COPY mysql-first-time.sql /docker-entrypoint-initdb.d/

