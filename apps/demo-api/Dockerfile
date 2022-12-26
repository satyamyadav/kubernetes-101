FROM node:14.19.3
# RUN mkdir -p /home/node/app && chown -R node:node /home/node/app
RUN mkdir -p /home/node/app && chown -R node:node /home/node/app
WORKDIR /home/node/app
# COPY --chown=node:node package*.json ./
RUN npx express-generator --view=ejs .
RUN npm i
# COPY --chown=node:node . .
EXPOSE 3000
USER node
CMD [ "./bin/www" ]