# steps followed in this build process are not to be used in actual or production build images. 
# steps like git clone and npm build should not be used in docker build process, this is used to ease the tutorial demo image build only.

FROM node:16.15.0
RUN mkdir -p /home/node/app && chown -R node:node /home/node/app
WORKDIR /home/node
# COPY --chown=node:node package*.json ./
RUN git clone --depth 1 https://github.com/tanem/express-bookshelf-realworld-example-app.git app
RUN chown -R node:node /home/node/app
WORKDIR /home/node/app
RUN npm i
# COPY --chown=node:node . .
EXPOSE 3000
USER node
CMD [ "./bin/start.sh" ]