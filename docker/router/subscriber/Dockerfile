FROM node:16-alpine as node

# ----------------------------------------
# Builds Docker container for router-subscriber package.


# ----------------------------------------

FROM node as build
RUN apk update
RUN apk add git

ENV HOME=/tmp/build \
    PATH=/tmp/build/node_modules/.bin:./node_modules/.bin:${PATH}

WORKDIR /tmp/build

ARG TEMP_DEPS_DIR

# ----- Copy only the files that affect yarn install -----
# Allows docker to use cache and skip install if dependencies are unchanged.
COPY .yarn /tmp/build/.yarn/
COPY .yarnrc.yml /tmp/build/
COPY package.json /tmp/build/
COPY packages/agents/router/package.json /tmp/build/packages/agents/router/
COPY packages/adapters/cache/package.json /tmp/build/packages/adapters/cache/
COPY packages/adapters/subgraph/package.json /tmp/build/packages/adapters/subgraph/
COPY packages/adapters/web3signer/package.json /tmp/build/packages/adapters/web3signer/
COPY packages/adapters/txservice/package.json /tmp/build/packages/adapters/txservice/
COPY packages/deployments/contracts/package.json /tmp/build/packages/deployments/contracts/
COPY packages/utils/package.json /tmp/build/packages/utils/
COPY yarn.lock /tmp/build/

# ----- Install dependencies -----
# Install dependencies exactly as in the yarn.lock file - no updates.
RUN yarn install

# ----- Copy source and all other files that affect lint, test, build -----
COPY config config/
COPY packages/agents/router /tmp/build/packages/agents/router
COPY packages/adapters/cache /tmp/build/packages/adapters/cache
COPY packages/adapters/subgraph /tmp/build/packages/adapters/subgraph
COPY packages/adapters/web3signer /tmp/build/packages/adapters/web3signer
COPY packages/adapters/txservice /tmp/build/packages/adapters/txservice
COPY packages/deployments/contracts /tmp/build/packages/deployments/contracts/
COPY packages/utils /tmp/build/packages/utils
COPY .eslintignore /tmp/build/
COPY .eslintrc.js /tmp/build/

# ----- Lint, test and build -----
RUN yarn build

# ----------------------------------------
# Copy files to the deployment image.
# ----------------------------------------

FROM node as runtime

ENV NODE_ENV=production \
    PORT=8080

ARG COMMIT_HASH
ENV COMMIT_HASH ${COMMIT_HASH:-unknown}

# ----- Copy files required at runtime by the app -----
COPY --from=build --chown=node:node /tmp/build /home/node

# This user is created in the base image with uid and gid = 1000.
USER node

WORKDIR /home/node/packages/agents/router

EXPOSE 8080

CMD ["node", "--trace-warnings", "dist/subscriber/index.js"]