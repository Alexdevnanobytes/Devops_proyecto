FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

ARG VITE_API_VENTAS
ENV VITE_API_VENTAS=$VITE_API_VENTAS
ARG VITE_API_DESPACHOS
ENV VITE_API_DESPACHOS=$VITE_API_DESPACHOS

RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

RUN printf 'server {\n\
    listen 80;\n\
    server_name _;\n\
    root /usr/share/nginx/html;\n\
    index index.html;\n\
\n\
    # 🔓 Permitir Cross-Origin para que se carguen imágenes de servidores externos\n\
    add_header Access-Control-Allow-Origin "*";\n\
    add_header Cross-Origin-Resource-Policy "cross-origin";\n\
\n\
    location / {\n\
        try_files $uri $uri/ /index.html;\n\
    }\n\
\n\
    location /api/ventas/ {\n\
        proxy_pass http://backend-service:8080/;\n\
        proxy_set_header Host $host;\n\
        proxy_set_header X-Real-IP $remote_addr;\n\
    }\n\
\n\
    location /api/despachos/ {\n\
        proxy_pass http://backend-despachos-service:8081/;\n\
        proxy_set_header Host $host;\n\
        proxy_set_header X-Real-IP $remote_addr;\n\
    }\n\
\n\
    location ~* \\.(?:ico|css|js|gif|jpe?g|png|woff2?)$ {\n\
        expires 6M;\n\
        access_log off;\n\
        add_header Cache-Control "public";\n\
        # Re-aplicamos las cabeceras aquí para que los estáticos locales no las pisen\n\
        add_header Access-Control-Allow-Origin "*";\n\
        add_header Cross-Origin-Resource-Policy "cross-origin";\n\
    }\n\
}\n' > /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]