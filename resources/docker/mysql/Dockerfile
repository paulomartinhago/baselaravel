FROM library/mysql:5.7

MAINTAINER Paulo Martinhago <pmartinhago@gmail.com>

ARG MYSQL_DATABASE=goutnix
ARG MYSQL_USER=goutnix
ARG MYSQL_PASSWORD=goutnix
ARG MYSQL_ROOT_PASSWORD=root

ENV MYSQL_DATABASE=$MYSQL_DATABASE
ENV MYSQL_USER=$MYSQL_USER
ENV MYSQL_PASSWORD=$MYSQL_PASSWORD
ENV MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD

# Laravel requirement for preventing migration timestamp incompatibility
COPY mode.cnf /etc/mysql/mysql.conf.d/mode.cnf