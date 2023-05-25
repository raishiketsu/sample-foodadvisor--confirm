FROM node:16

WORKDIR /app

ADD ./package.json /app

ADD ./yarn.lock /app

ADD ./.env.development /app

RUN yarn install --frozen-lockfile

ADD . /app

EXPOSE 3000

CMD ["yarn", "dev"]