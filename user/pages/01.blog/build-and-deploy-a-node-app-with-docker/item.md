---
title: Build and Deploy a Node App with Docker
date: 17:34 06/18/2016
taxonomy:
    category: blog
    tag: [javascript, node, docker]
---
How many times have you deployed your app that was working perfectly in your local environment to production, only to see it break? Whether it was directly related to the bug or feature you were working on, or another random issue entirely, this happens all too often for most developers. Errors like this not only slow you down, they're also embarrassing.

Why does this happen? Usually, it's because your development environment on your local machine is different from the production environment you're deploying to. The 10th factor of the [Twelve-Factor App](http://12factor.net/) is [Dev/prod parity](http://12factor.net/dev-prod-parity). This means that your development, staging, and production environments should be as similar as possible. The authors of the Twelve-Factor App spell out three "gaps" that can be present. They are:

* The time gap: A developer may work on code that takes days, weeks, or even months to go into production.
* The personnel gap: Developers write code, ops engineers deploy it.
* The tools gap: Developers may be using a stack like Nginx, SQLite, and OS X, while the production deploy uses Apache, MySQL, and Linux.  
[Source](http://12factor.net/dev-prod-parity)

In this article, we will mostly focus on the tools gap, and how to bridge that gap in a Node application with Docker.

### The Tools Gap
In the Node ecosystem, the tools gap usually manifests itself either in differences in Node and npm versions, or differences in package dependency versions. If a package author publishes a breaking change in one of your dependencies or your dependencies' dependencies, it is entirely possible that your app will break on the next deployment (assuming you reinstall dependencies with `npm install` on every deployment), while it runs perfectly on your local machine.

Although you can work around this issue using tools like `npm shrinkwrap`, adding Docker to the mix will streamline your deployment lifecycle, and minimize broken deployments to production.

### Why Docker?
Docker is unique because it can be used the same way in development in production. When you enable the architecture of your app to be able to run inside containers, you can easily scale out and create small containers that can be composed together to make one awesome system.  Then, you can mimic this architecture in development so you never have to guess how your app will behave in production. In regards to the time gap and the personnel gap, Docker makes it easier for developers to automate deployments thereby decreasing time to production and making it easier for full stack teams to own deployments.

### Tools and Concepts
When developing inside Docker containers, the two most important concepts are `docker-compose` and volumes.

`docker-compose` helps define mulit-container environments and the ability to run them with one command.

Here are some of the more often used `docker-compose` commands:

* `docker-compose build` - Builds images for services defined in the docker-compose.yml.
* `docker-compose up` - Create and start services. This is the same as running `docker-compose create && docker-compose start`
* `docker-compose run` - Run a one-off command inside a container

Volumes allow you to mount files from the host machine into the container. When the files on your host machine change, they change inside the container as well. This is important so that we don't have to constantly rebuild containers during development every time we make a change. You can also use a tool like [node-mon](https://github.com/remy/nodemon) to automatically restart the node app on changes.

Let's walkthrough some tips and tricks with developing Node apps inside Docker containers.

#### Setup Dockerfile and docker-compose.yml

When you start a new project with Docker, you'll first want to define a barebones `Dockerfile` and `docker-compose.yml` to get you started. Here's an example `Dockerfile`:

    FROM node:6.2.1
    RUN useradd --user-group --create-home --shell /bin/false app-user
    ENV HOME=/home/app-user
    USER app-user
    WORKDIR $HOME/app

This Dockerfile displays two best practices:

1. Favor exact version tags over floating tags such as `latest`. Node releases often these days, and you don't want to implicitly upgrade when building your container on another machine. By specifying a version such as 6.2.1, you ensure that anyone who builds the image will always be working from the same node version.  

2. Create a new user to run the app inside the container. Without this step, everything would run under root in the container. You certainly wouldn't do that on a physical machine, so don't do in Docker containers either.

Here's an example starter `docker-compose.yml`:

    web:
      build: .
      volumes:
        - .:/home/app-user/app

Pretty simple right? Here we are telling Docker to build the web service based on our Dockerfile, and create a volume from our current host directory to `/home/app-user/app` inside the container.

This simple setup lets us build our container with `docker-compose build` and then run bash inside it with `docker-compose run --rm web /bin/bash`. Now, it's essentially the same as if you were SSH'd in to a remote server, or working off a VM except that any file you create inside the container will be on your host machine and vice versa.

With that in mind, you can bootstrap your Node app from inside your container using `npm init -y` and `npm shrinkwrap`. Then, you could install any modules you need such as Express.

#### Install node modules on build

With that done, we need to update our Dockerfile to install dependencies from npm when the image is built. Here is the updated `Dockerfile`:

    FROM node:6.2.1
    RUN useradd --user-group --create-home --shell /bin/false app-user
    ENV HOME=/home/app-user

    COPY package.json npm-shrinkwrap.json $HOME/app/
    RUN chown -R app-user:app-user $HOME/*

    USER app-user
    WORKDIR $HOME/app
    RUN npm install

Notice that we had to change ownership of the copied files to app-user. This is because files copied into a container are automatically owned by root.

#### Add a volume for the node_modules directory

We also need to make an update to our docker-compose.yml, to make sure that our modules are installed inside the container properly.

    web:
      build: .
      volumes:
        - .:/home/app-user/app
        - /home/app-user/app/node_modules

Without adding a data volume to `/home/app-user/app/node_modules`, the `node_modules` wouldn't exist at runtime in the container because our host directory, which won't contain the `node_modules` directory, would be mounted and hide the `node_modules` directory that was created when the container was built. For more information, see [this Stack Overflow post](http://stackoverflow.com/questions/30043872/docker-compose-node-modules-not-present-in-a-volume-after-npm-install-succeeds).

#### Running your app
Once you've got an entry point to your app ready to go, simply add it as a `CMD` in your Dockerfile:

`CMD ["node", "index.js"]`

This will automatically start our app on `docker-compose up`.

Running tests inside your container is easy as well.  
`docker-compose --rm run web npm test`

You could easily hook this into CI as well.

### Production

Now going to production with your Docker powered Node app is a breeze! Just use `docker-compose` again. You will probably want to define another `docker-compose.yml` that is especially written for production use. This means removing volumes, binding to different ports, setting `NODE_ENV=production`, etc. Once you have a production config file, you can tell `docker-compose` to use it like so:

`docker-compose -f docker-compose.yml -f docker-compose.production.yml up`

The `-f` lets you specify a list of files that are merged in the order specified.

Here is a complete `Dockerfile` and `docker-compose.yml` for reference:

    # Dockerfile
    FROM node:6.2.1

    RUN useradd --user-group --create-home --shell /bin/false app-user

    ENV HOME=/home/app-user

    COPY package.json npm-shrinkwrap.json $HOME/app/
    RUN chown -R app-user:app-user $HOME/*

    USER app-user
    WORKDIR $HOME/app
    RUN npm install

    CMD ["node", "index.js"]

    # docker-compose.yml
    web:
      build: .
      ports:
        - '3000:3000'
      volumes:
        - .:/home/app-user/app
        - /home/app-user/app/node_modules
