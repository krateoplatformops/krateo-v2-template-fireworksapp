FROM bitnami/nginx
LABEL maintainer "Krateo <contact@krateoplatformops.io>"

COPY index.html /app

ENV PORT 8080
EXPOSE 8080

COPY nginx.conf /opt/bitnami/nginx/conf/bitnami/location.conf
CMD ["nginx", "-g", "daemon off;"]