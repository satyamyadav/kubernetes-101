# steps followed in this build process are not to be used in actual or production build images. 
# steps like git clone and npm build should not be used in docker build process, this is used to ease the tutorial demo image build only.

FROM node:14.19.3
RUN mkdir -p /home/node/app && chown -R node:node /home/node/app
WORKDIR /home/node/app
RUN npx express-generator --view=ejs .
RUN npm i
EXPOSE 3000
USER node
CMD [ "./bin/www" ]
