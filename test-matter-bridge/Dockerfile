# docker build -t test-matter-bridge .
# docker run --rm -p 5540:5540/tcp -p 5540:5540/udp test-matter-bridge

#ARG BUILD_FROM
#FROM $BUILD_FROM

ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base
FROM ghcr.io/home-assistant/amd64-base

# install node and npm
ARG NODE_VERSION="18.18.2-r0"
ARG NPM_VERSION="9.6.6-r0"
RUN apk update && apk add nodejs="$NODE_VERSION" npm="$NPM_VERSION" icu-data-full

# log install packages versions
RUN node -v
RUN npm -v

# expose web interface port not needed with ingress
#EXPOSE 3000
# expose ports in both protocoles
EXPOSE 5540/tcp
EXPOSE 5540/udp
# expose ports in both protocoles mdns
EXPOSE 5353/tcp
EXPOSE 5353/udp

RUN npm i -g @project-chip/matter-node.js-examples

# Start the server using the production build
CMD [ "matter-bridge", "-store", "/data", "-passcode", "20202023", "-discriminator", "3621" ]
