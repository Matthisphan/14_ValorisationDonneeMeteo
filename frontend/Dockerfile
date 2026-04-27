ARG FRONTEND_BUILD_IMAGE=node:24-bookworm
ARG FRONTEND_RUNTIME_IMAGE=node:24-alpine

FROM ${FRONTEND_BUILD_IMAGE} AS build

WORKDIR /app

COPY package*.json ./

RUN ["node", "--version"]
RUN ["npm", "--version"]

RUN ["npm", "install"]

COPY . .

ENV NODE_OPTIONS=--max-old-space-size=4096

RUN ["npm", "run", "build"]

FROM ${FRONTEND_RUNTIME_IMAGE} AS prod

WORKDIR /app

COPY --from=build /app/.output /app/.output

ENV NODE_ENV=production

EXPOSE 3000

CMD ["node", ".output/server/index.mjs"]
