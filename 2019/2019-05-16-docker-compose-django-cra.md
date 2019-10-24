---
title: Creating an app with Docker Compose, Django, and Create React App
published: true
description: How to containerize independent front-end and back-end services while still getting them to play nice with each other
series: Docker, Django, and React
tags: #django, #docker, #react
---

Final code for this tutorial if you want to skip the text, or get lost with some of the references, can be found on [GitHub](https://github.com/cfranklin11/docker-django-react).

Inspired by sports data sites like [Squiggle](https://squiggle.com.au/) and [Matter of Stats](http://www.matterofstats.com/), in building the app that houses [Tipresias](https://github.com/cfranklin11/tipresias) (my footy-tipping machine-learning model), I wanted to include a proper front-end with metrics, charts, and round-by-round tips. I already knew that I would have to dockerize the thing, because I was working with multiple packages across Python and R, and such complex dependencies are incredibly difficult to manage in a remote-server context (and impossible to run on an out-of-the-box service like Heroku) without using Docker. I could have avoided exacerbating my complexity issue by using basic Django views (i.e. static HTML templates) to build my pages, but having worked with a mishmash of ancient Rails views that had React components grafted on to add a little interactivity (then a lot of interactivity), I preferred starting out with clear separation between my frontend and backend. What's more, I wanted to focus on the machine learning, data engineering, and server-side logic (not to mention the fact that I couldn't design my way out of a wet paper bag), so my intelligent, lovely wife agreed to help me out with the frontend, and there was no way she was going to settle for coding in the context of a 10-year-old paradigm. It was going to be a modern web-app architecture, or I was going to have to pad my own divs.

The problem with combining Docker, Django, and React was that I had never set up anything like this before, and, though I ultimately figured it out, I had to piece together my solution from multiple guides/tutorials that did some aspect of what I wanted without covering the whole. In particular, the tutorials that I found tended to build static Javascript assets that Django could use in its views. This is fine for production, but working without hot-reloading (i.e. having file changes automatically restart the server, so that they are reflected in the relevant pages that are loaded in the browser) is the hair shirt of development: at first you think you can endure the mild discomfort, but the constant itching wears you down, becoming the all-consuming focus of your every waking thought, driving you to distraction, to questioning all of your choices in life. Imagine having to run a build command that takes maybe a minute every time you change so much as a single line of code. Side projects don't exactly require optimal productivity, but, unlike jobs, if they become a pain to work on, it's pretty easy to just quit.

## What we're gonna do

1. Create a Django app that runs inside a Docker container.
2. Create a React app with the all-too-literally-named Create React App that runs inside a Docker container.
3. Implement these dockerized apps as services in Docker Compose.
4. Connect the frontend service to a basic backend API from which it can fetch data.

**Note:** This tutorial assumes working knowledge of Docker, Django, and React in order to focus on the specifics of getting these three things working together in a dev environment.

## 1. Create a dockerized Django app

Let's start by creating a project directory named whatever you want, then a `backend` subdirectory with a `requirements.txt` that just adds the `django` package for now. This will allow us to install and run Django in a Docker image built with the following `Dockerfile`:

```
# Use an official Python runtime as a parent image
FROM python:3.6

# Adding backend directory to make absolute filepaths consistent across services
WORKDIR /app/backend

# Install Python dependencies
COPY requirements.txt /app/backend
RUN pip3 install --upgrade pip -r requirements.txt

# Add the rest of the code
COPY . /app/backend

# Make port 8000 available for the app
EXPOSE 8000

# Be sure to use 0.0.0.0 for the host within the Docker container,
# otherwise the browser won't be able to find it
CMD python3 manage.py runserver 0.0.0.0:8000
```

In the terminal, run the following commands to build the image, create a Django project named hello_world, and run the app:

```bash
docker build -t backend:latest backend
docker run -v $PWD/backend:/app/backend backend:latest django-admin startproject hello_world .
docker run -v $PWD/backend:/app/backend -p 8000:8000 backend:latest
```

Note that we create a volume for the `backend` directory, so the code created by `startproject` will appear on our machine. The `.` at the end of the create command will place all of the Django folders and files inside our backend directories instead of creating a new project directory, which can complicate managing the working directory within the Docker container.

Open your browser to `localhost:8000` to verify that the app is up and running.

## 2. Create a dockerized Create React App (CRA) app

Although I got my start coding frontend Javascript, I found my calling working on back-end systems. So, through a combination of my own dereliction and the rapid pace of change of frontend tools and technologies, I am ill-equipped to set up a modern frontend application from scratch. I am, however, fully capable of installing a package and running a command.

Unlike with the Django app, we can't create a Docker image with a CRA app all at once, because we'll first need a `Dockerfile` with node, so we can initialise the CRA app, then we'll be able to add the usual `Dockerfile` commands to install dependencies. So, create a `frontend` directory with a `Dockerfile` that looks like the following:

```
# Use an official node runtime as a parent image
FROM node:8

WORKDIR /app/

# Install dependencies
# COPY package.json yarn.lock /app/

# RUN npm install

# Add rest of the client code
COPY . /app/

EXPOSE 3000

# CMD npm start
```

Some of the commands are currently commented out, because we don't have a few of the files referenced, but we will need these commands later. Run the following commands in the terminal to build the image, create the app, and run it:

```bash
docker build -t frontend:latest frontend
docker run -v $PWD/frontend:/app frontend:latest npx create-react-app hello-world
mv frontend/hello-world/* frontend/hello-world/.gitignore frontend/ && rmdir frontend/hello-world
docker run -v $PWD/frontend:/app -p 3000:3000 frontend:latest npm start
```

Note that we move the newly-created app directory's contents up to the frontend directory and remove it. Django gives us the option to do this by default, but I couldn't find anything to suggest that CRA will do anything other than create its own directory. Working around this nested structure is kind of a pain, so I find it easier to just move everything up the docker-service level and work from there. Navigate your browser to `localhost:3000` to make sure the app is running. Also, you can uncomment the rest of the commands in the `Dockerfile`, so that any new dependencies will be installed the next time you rebuild the image.

## 3. Docker-composify into services

Now that we have our two Docker images and are able to run the apps in their respective Docker containers, let's simplify the process of running them with Docker Compose. In `docker-compose.yml`, we can define our two services, `frontend` and `backend`, and how to run them, which will allow us to consolidate the multiple `docker` commands, and their multiple arguments, into much fewer `docker-compose` commands. The config file looks like this:

```yaml
version: "3.2"
services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app/backend
    ports:
      - "8000:8000"
    stdin_open: true
    tty: true
    command: python3 manage.py runserver 0.0.0.0:8000
  frontend:
    build: ./frontend
    volumes:
      - ./frontend:/app
      # One-way volume to use node_modules from inside image
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    depends_on:
      - backend
    command: npm start
```

We've converted the various arguments for the docker commands into key-value pairs in the config file, and now we can run both our frontend and backend apps by just executing `docker-compose up`. With that, you should be able to see them both running in parallel at `localhost:8000` and `localhost:3000`.

## 4. Connecting both ends into a single app

Of course, the purpose of this post is not to learn how to overcomplicate running independent React and Django apps just for the fun of it. We are here to build a single, integrated app with a dynamic, modern frontend that's fed with data from a robust backend API. Toward that goal, while still keeping the app as simple as possible, let's have the frontend send text to the backend, which will return a count of the number of characters in the text, which the frontend will then display.

### Setting up the Django API

Let's start by creating an API route for the frontend to call. You can create a new Django app (which is kind of a sub-app/module within the Django project architecture) by running the following in the terminal:

`docker-compose run --rm backend python3 manage.py startapp char_count`

This gives you a new directory inside `backend` called `char_count`, where we can define routes and their associated logic.

We can create the API response in `backend/char_count/views.py` with the following, which, as promised, will return the character count of the submitted text:

```python
from django.http import JsonResponse


def char_count(request):
    text = request.GET.get("text", "")

    return JsonResponse({"count": len(text)})
```

Now, to make the Django project aware of our new app, we need to update `INSTALLED_APPS` in `backend/hello_world/settings.py` by adding `"char_count.apps.CharCountConfig"` to the list. To add our count response to the available URLs, we update `backend/hello_world/urls.py` with our char_count view as follows:

```python
from django.contrib import admin
from django.urls import path
from char_count.views import char_count

urlpatterns = [
    path('admin/', admin.site.urls),
    path('char_count', char_count, name='char_count'),
]
```

Since we're changing project settings, we'll need to stop our Docker Compose processes (either ctl+c or `docker-compose stop` in a separate tab) and start it again with `docker-compose up`. We can now go to `localhost:8000/char_count?text=hello world` and see that it has 11 characters.

### Connecting React to the API

First, let's add a little more of that sweet config to make sure we don't get silent errors related to networking stuff that we'd really rather not deal with. Our Django app currently won't run on any host other than `localhost`, but our React app can only access it via the Docker service name `backend` (which does some magic host mapping stuff). So, we need to add `"backend"` to `ALLOWED_HOSTS` in `backend/hello_world/settings.py`, and we add `"proxy": "http://backend:8000"` to `package.json`. This will allow both services to talk to each other. Also, we'll need to use the npm package `axios` to make the API call, so add it to `package.json` and rebuild the images with the following:

```bash
docker-compose run --rm frontend npm add axios
docker-compose down
docker-compose up --build
```

My frontend dev skills are, admittedly, subpar, but please keep in mind that the little component below is not a reflection of my knowledge of React (or even HTML for that matter). In the interest of simplicity, I just removed the CRA boilerplate and replaced it with an input, a button, a click handler, and a headline.

```javascript
import React from "react";
import axios from "axios";
import "./App.css";

function handleSubmit(event) {
  const text = document.querySelector("#char-input").value;

  axios
    .get(`/char_count?text=${text}`)
    .then(({ data }) => {
      document.querySelector(
        "#char-count"
      ).textContent = `${data.count} characters!`;
    })
    .catch(err => console.log(err));
}

function App() {
  return (
    <div className="App">
      <div>
        <label htmlFor="char-input">How many characters does</label>
        <input id="char-input" type="text" />
        <button onClick={handleSubmit}>have?</button>
      </div>

      <div>
        <h3 id="char-count"></h3>
      </div>
    </div>
  );
}

export default App;
```

Now, when we enter text into the input and click the button, the character count of the text is displayed below. And best of all: we got hot reloading all up and down the field! You can add new components to the frontend, new classes to the backend, and all your changes (short of config or dependencies) will be reflected in the functioning of the app as you work, without having to manually restart the servers.

## Summary

In the end, setting all this up isn't too complicated, but there are lots of little gotchas, many of which don't give you a nice error message to look up on Stack Overflow. Also, at least in my case, I really struggled at first to conceptualise how the pieces were going to work together. Would the React app go inside the Django app, like it does with `webpacker` in Rails? If the two apps are separate Docker Compose services, how do you connect them? In the end we learned how to:

- Set up Django in a Docker container.
- Set up Create React App in a Docker container
- Configure those containers with Docker Compose
- Use Docker Compose's service names (e.g. `backend`) and `package.json`'s `"proxy"` attribute to direct React's HTTP call to Django's API and display the response.
