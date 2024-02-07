# Use an official Apache image as a base image
FROM httpd:2.4

# Copy your custom Apache configuration into the container
COPY apache-reverse-proxy.conf /usr/local/apache2/conf/httpd.conf

# Copy your index.html file to the default document root
COPY index.html /usr/local/apache2/htdocs/
