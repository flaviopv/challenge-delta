FROM mysql:5.6
ENV MYSQL_DATABASE packages
RUN apt-get update -y
COPY ./db/ /docker-entrypoint-initdb.d/
