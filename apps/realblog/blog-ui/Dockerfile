# steps followed in this build process are not to be used in actual or production build images. 
# steps like git clone and npm build should not be used in docker build process, this is used to ease the tutorial demo image build only. 

FROM node:14.19.3 as base

RUN mkdir -p /home/node/app && chown -R node:node /home/node/app
WORKDIR /home/node
RUN git clone --depth 1 https://github.com/angelguzmaning/ts-redux-react-realworld-example-app.git app
RUN chown -R node:node /home/node/app
WORKDIR /home/node/app
RUN npm i
RUN npm run build
USER node

FROM nginx

COPY --from=base /home/node/app/build /usr/share/nginx/html
