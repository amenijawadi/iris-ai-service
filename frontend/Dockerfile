# Étape 1 : Build du frontend avec Node.js 
FROM node:20-alpine AS build
WORKDIR /app

# Copier les fichiers de dépendances et installer
COPY package*.json ./
RUN npm install

# Copier tout le code source
COPY . .

# Construire le projet
RUN npm run build

# Étape 2 : Nginx pour servir les fichiers
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
