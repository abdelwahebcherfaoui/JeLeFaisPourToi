# syntax=docker/dockerfile:1

# --- Étape 1 : construire le site avec Node ---
FROM node:22-alpine AS build
WORKDIR /app

COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm npm ci

COPY . .
# Vide = le front appelle l'API sur la même adresse que lui (/api/...),
# et c'est Nginx qui fait suivre. Plus besoin de CORS.
ARG VITE_API_BASE_URL=""
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL
RUN npm run build

# --- Étape 2 : servir le site avec Nginx ---
# Variante "unprivileged" : Nginx tourne sans les droits root, sur le port 8080.
FROM nginxinc/nginx-unprivileged:1.29-alpine

COPY nginx/default.conf.template /etc/nginx/templates/default.conf.template
COPY --from=build /app/dist /usr/share/nginx/html

# Adresse de l'API, remplaçable au lancement (docker compose, Cloud Run...).
ENV API_URL=http://api:8080
EXPOSE 8080