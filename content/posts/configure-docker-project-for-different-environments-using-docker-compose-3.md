---
title: Configure Docker project for different environments using Docker Compose 3
date: 2018-09-07
---

![docker compose](/img/docker-compose.png)

We all know that Docker is awesome, but when it comes to separating deployment configs, there is little to no strict guideline. In this article, I will try to explain one of the most popular techniques which is using `docker-compose` with environment variables. There is also an example with Node.js at the end.

<!--more-->

## Why do we even need different configuration for each environment?

In a software development lifecycle, there may be as little deployment environments as just development and production. However, there may also be as many as development, integration, testing, staging and production. These environments are different by nature and thus require some slight (or considerable) change of factors.

According to [The Twelve Factors](https://12factor.net/config), config should live outside *codebase*, recommendedly inside [_environment variables_](https://en.wikipedia.org/wiki/Environment_variable?oldformat=true). One common rule of thumb is that your config is correctly isolated from your codebase if and only if your codebase is ready to be made public without security compromises.

> Config is correctly isolated from codebase if and only if the codebase is ready to be made public without security compromises

## Meet the Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) is a Docker companion tool used to coordinate multiple containers with configurations. In other words, instead of building a bunch of images (app, database, Redis,...) and linking them together with a bunch of arguments, Compose will only need you one file `docker-compose.yaml` which defines everything from build-time to run-time and one command `docker-compose up`. I won't go too deep into Docker Compose so if this is your first time with it, try reading other tutorials for your tech stack and come back after you have got your `docker-compose.yml` right.

The wonderful things about Compose is that firstly, it can [use environment variables](https://docs.docker.com/compose/environment-variables/) from shell or `.env` file and secondly, it can merge multiple Compose file into one. The latter is great because we can just define the base config in one file and deployment-specific config in other files without repeating the whole thing. [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself?oldformat=true), period.

The syntax for merging Compose file is like this:

```bash
docker-compose -f docker-compose.yml -f docker-compose.second.yml -f docker-compose.third.yml up
```

The team behind Compose probably thought that this command was too long and so has provided us with a little nice hack: `docker-compose up` will automatically merge `docker-compose.yml` with another file called `docker-compose.override.yml`. As such, you can designate the *override *file for development or production config depending on where you want to type less. However, in my opinion, creating a convention like that is kind of ambiguous and should be avoided, *override* file is better served as a local config and hence be included in `.gitignore` as well.

But, but, what shall be the differences between environments? Here are some of my suggestions:

- Database and third-party API credentials
- Binding to different ports on the host
- Different environment variables for your app like `NODE_ENV` or logging level
- Development need mounting volume(s) to reflect source code changes to Docker container
- Production probably need a restart policy like `restart: always` to avoid downtime
- Node.js project might need to run `npm install` with `--only=production` option

## Dealing with Dockerfile

Occasionally you might want to run different commands in `Dockerfile` based on environments, `npm install` for example. We probably know that bash script has conditional so we could come up with something like this:

```docker
RUN if [ "$NODE_ENV" = "development" ]; \
  then npm install; \
  else npm install --only=production; \
  fi
```

## Example code

Here is a short example of a Docker Node.js project with two environments development and production.

`/Dockerfile`

```docker
FROM node:8-alpine

WORKDIR /usr/src/your-app

COPY package*.json ./

RUN if [ "$NODE_ENV" = "development" ]; \
	then npm install;  \
	else npm install --only=production; \
	fi

COPY . .
```

`/docker-compose.yaml`

```yaml
version: "3"
services:
  app:
    build: .
    ports:
      - "${PORT}:80"
```

`/docker-compose.dev.yaml`

```yaml
version: "3"
services:
  app:
    command: npm run dev
    volumes:
      - .:/usr/src/your-app
    environment:
      - NODE_ENV=development
```

`/docker-compose.prod.yaml`

```yaml
version: "3"
services:
  app:
    command: npm run prod
    restart: always
    environment:
      - NODE_ENV=production
```

Development command:

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

Production command:

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

If the commands is too verbose for you, you could always store them in a bash script and run it instead.

## Key takeaways

- Store deployment configs in environment variables
- Leverage Compose file merging feature to keep configs DRY

Feel free to leave your comments below and hit clap if you find this piece of writing useful! Thank you for reading 😍.
