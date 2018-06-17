FROM node
ADD . /usr/src/twt-app
WORKDIR /usr/src/twt-app
RUN npm install hapi
EXPOSE 3000
CMD [ "node", "app.js" ]