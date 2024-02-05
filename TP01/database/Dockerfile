FROM postgres:14.1-alpine

ADD CreateScheme.sql /docker-entrypoint-initdb.d
ADD InsertData.sql /docker-entrypoint-initdb.d
