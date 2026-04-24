# Images configurables pour basculer vers des variantes durcies (DHI).
# Note: Utilise node:24-bookworm au lieu d'alpine pour plus de stabilité lors du build
# Alpine a des ressources mémoire limitées et peut causer des crashes pendant le build Nuxt
ARG FRONTEND_BUILD_IMAGE=node:24-bookworm
ARG FRONTEND_RUNTIME_IMAGE=node:24-alpine

# Etape de build: compile l'application Nuxt.
FROM ${FRONTEND_BUILD_IMAGE} AS build

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

# Augmente la limite du heap Node.js pour éviter l'erreur "out of memory" lors du build Nuxt
# 4096 MB = 4 GB pour accommoder les builds complexes
ENV NODE_OPTIONS=--max-old-space-size=4096

RUN npm run build

# Etape runtime: image finale minimale avec seulement l'artefact .output.
FROM ${FRONTEND_RUNTIME_IMAGE} AS prod

WORKDIR /app

COPY --from=build /app/.output /app/.output

ENV NODE_ENV=production

EXPOSE 3000

CMD [ "node", ".output/server/index.mjs" ]
