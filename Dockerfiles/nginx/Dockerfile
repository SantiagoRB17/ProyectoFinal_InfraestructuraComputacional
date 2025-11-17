FROM nginx:stable

ENV TZ=America/Bogota

RUN apt-get update && apt-get install -y nano curl && rm -rf /var/lib/apt/lists/*

COPY ./html/ /usr/share/nginx/html/

EXPOSE 80