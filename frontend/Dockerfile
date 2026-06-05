# Stage 1: build React
FROM node:20-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

# Stage 2: Nginx serve static files
FROM nginx:1.27-alpine

# Update vulnerable Alpine packages in runtime image
RUN apk update && apk upgrade --no-cache libcrypto3 libssl3

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
