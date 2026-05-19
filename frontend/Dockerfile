FROM alpine:latest AS assets
WORKDIR /dist
COPY index.html app.js default.conf ./

FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=assets /dist/index.html /dist/app.js /usr/share/nginx/html/
COPY --from=assets /dist/default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80