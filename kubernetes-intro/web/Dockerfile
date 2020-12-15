FROM nginx:latest
RUN usermod -u 1001 nginx
EXPOSE 8000
WORKDIR /app
COPY ./default.conf /etc/nginx/conf.d/
