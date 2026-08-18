FROM nginxinc/nginx-unprivileged:1.31-alpine3.24

LABEL org.opencontainers.image.source="https://github.com/jedmond1971/Chameleon"
LABEL org.opencontainers.image.description="A small, private, entirely client-side image format converter"
LABEL org.opencontainers.image.licenses="MIT"

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY chameleon.html /usr/share/nginx/html/chameleon.html

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s CMD wget -q -O- http://127.0.0.1:8080/chameleon.html > /dev/null || exit 1
