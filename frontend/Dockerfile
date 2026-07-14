# Build Stage
# node:18-alpine is used because react-scripts 3.x bundles webpack 4,
# which uses MD4 hashes unsupported by OpenSSL 3 (Node 17+).
# NODE_OPTIONS=--openssl-legacy-provider re-enables the legacy provider.
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

ENV NODE_OPTIONS=--openssl-legacy-provider

RUN npm run build

# Production Stage
FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]