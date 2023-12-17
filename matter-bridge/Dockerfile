# docker build -t ha-matter-bridge-addon .
# docker run --rm --network host ha-matter-bridge-addon

#ARG BUILD_FROM
#FROM $BUILD_FROM as base-node

ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base
FROM ghcr.io/home-assistant/amd64-base as base-node

# install node and npm
#ARG NODE_VERSION="18.18.2-r0"
#ARG NPM_VERSION="9.6.6-r0"
RUN #apk update && apk add nodejs="$NODE_VERSION" npm="$NPM_VERSION" icu-data-full
RUN apk update && apk add nodejs npm icu-data-full

# log install packages versions
RUN node -v
RUN npm -v

###################
# BUILD FOR LOCAL DEVELOPMENT
###################

FROM base-node As development

# Create app directory
WORKDIR /usr/src/app

# Copy application dependency manifests to the container image.
# A wildcard is used to ensure copying both package.json AND package-lock.json (when available).
# Copying this first prevents re-running npm install on every code change.
COPY src/ha-matter-bridge/package*.json ./

# Install app dependencies using the `npm ci` command instead of `npm install`
RUN npm ci

# Bundle app source
COPY src/ha-matter-bridge .

###################
# BUILD FOR PRODUCTION
###################

FROM base-node As build

WORKDIR /usr/src/app

COPY src/ha-matter-bridge/package*.json ./

# In order to run `npm run build` we need access to the Nest CLI which is a dev dependency. In the previous development stage we ran `npm ci` which installed all dependencies, so we can copy over the node_modules directory from the development image
COPY --from=development /usr/src/app/node_modules ./node_modules

COPY src/ha-matter-bridge .

# Run the build command which creates the production bundle
# RUN npm run build
RUN npx nx run ha-matter-bridge:build:production

# Set NODE_ENV environment variable
ENV NODE_ENV production

# Running `npm ci` removes the existing node_modules directory and passing in --only=production ensures that only the production dependencies are installed. This ensures that the node_modules directory is as optimized as possible
RUN npm ci --only=production && npm cache clean --force

###################
# PRODUCTION
###################

FROM base-node As production

# not port exposed, should use host network
# expose web interface port not needed with ingress
#EXPOSE 3000
# expose ports in both protocoles
#EXPOSE 5540/tcp
#EXPOSE 5540/udp
## expose ports in both protocoles mdns
#EXPOSE 5353/tcp
#EXPOSE 5353/udp

WORKDIR /usr/src/app

# Copy the bundled code from the build stage to the production image
COPY --from=build /usr/src/app/node_modules ./node_modules
COPY --from=build /usr/src/app/dist/apps/ha-matter-bridge ./dist

# Copy run script
COPY run.sh /
RUN chmod a+x /run.sh

# todo remove and use log level addon options for log
ENV NODE_ENV development

# Start the server using the production build
#CMD [ "node", "dist/main.js" ]
#CMD ["sh", "-c", "node dist/main.js --config /data/options.json --store /data/.device-node --hassUrl http://supervisor --hassAccessToken $SUPERVISOR_TOKEN --addon true" ]
CMD [ "/run.sh" ]