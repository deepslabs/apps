FROM ubuntu:24.04 as builder

# Install any needed packages
RUN apt-get update && \
  apt-get install --no-install-recommends -y build-essential curl git gnupg ca-certificates python3

# install nodejs
RUN curl -sL https://deb.nodesource.com/setup_lts.x | bash -
RUN apt-get install --no-install-recommends -y nodejs && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*
RUN npm install yarn -g

WORKDIR /apps
COPY . .

# Electron is only needed for desktop packaging, not the web build.
# Skip the ~120MB binary download which hangs/fails in restricted networks
# and aborts `yarn install` with YN0009.
ENV ELECTRON_SKIP_BINARY_DOWNLOAD=1

RUN yarn && NODE_ENV=production yarn build:www

# ===========================================================
FROM nginx:stable-alpine

# The following is mainly for doc purpose to show which ENV is supported
ENV WS_URL=

WORKDIR /usr/share/nginx/html

COPY docker/env.sh .

RUN apk add --no-cache bash; chmod +x env.sh

COPY docker/nginx.conf /etc/nginx/nginx.conf
COPY --from=builder /apps/packages/apps/build /usr/share/nginx/html

EXPOSE 80

CMD ["/bin/bash", "-c", "/usr/share/nginx/html/env.sh && nginx -g \"daemon off;\""]
