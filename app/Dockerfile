FROM nginx:alpine
ARG APP_VERSION=0.0.0
COPY index.html /usr/share/nginx/html/index.html
RUN sed -i "s/{{VERSION}}/${APP_VERSION}/g" /usr/share/nginx/html/index.html
