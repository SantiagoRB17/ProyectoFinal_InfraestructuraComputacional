FROM httpd:2.4

# Configurar zona horaria
ENV TZ=America/Bogota

# Instalar utilidades básicas
RUN apt-get update && apt-get install -y nano curl && rm -rf /var/lib/apt/lists/*

# Copiar archivos HTML personalizados
COPY ./html/ /usr/local/apache2/htdocs/

EXPOSE 80