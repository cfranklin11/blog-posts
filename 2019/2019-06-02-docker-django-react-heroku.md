---
title: Docker, Django, React: Building Assets and Deploying to Heroku
published: true
description: Second part in a series on working with Docker, Django, & React: now deployed to Heroku, with WhiteNoise serving assets.
tags: #docker, #django, #react, #heroku
series: Docker, Django, and React
---

Part 2 in a series on combining Docker, Django, and React. This builds on the development setup from [Part 1](https://dev.to/englishcraig/creating-an-app-with-docker-compose-django-and-create-react-app-31lf), so you might want to take a look at that first. If you want to skip to the end or need a reference, you can see the final version of the code on the [`production-heroku`](https://github.com/cfranklin11/docker-django-react/tree/production-heroku) branch of the repo.

---

Now that we have our app humming like a '69 Mustang Shelby GT500 in our local environment, doing hot-reloading donuts all over the parking lot, it's time to deploy that bad boy, so the whole world can find out how many characters there are in all their favourite phrases. In order to deploy this app to production, we'll need to do the following:

- Set up Django to use WhiteNoise to serve static assets in production.
- Create a production `Dockerfile` that combines our frontend and backend into a single app.
- Create a new Heroku app to deploy to.
- Configure our app to deploy a Docker image to Heroku.

## Use WhiteNoise to serve our frontend assets

### Update settings for different environments

Since we only want to use WhiteNoise in production, we'll have to change how our Django app's settings work to differentiate between the dev and prod environments. There are different ways to do this, but the one that seems to offer the most flexibility, and has worked well enough for me, is to create a settings file for each environment, all of which inherit from some base settings, then determine which settings file to use with an environment variable. In the `backend/hello_world`, which is our project directory, create a `settings` folder (as usual, with a `__init__.py` inside to make it a module), move the existing `settings.py` into it, and rename it `base.py`. This will be the collection of base app settings that all environments will inherit. To make sure we don't accidentally deploy with unsafe settings, cut the following code from `base.py`, and paste it into a newly-created `development.py`:

```python
# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/2.2/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = "<some long series of letters, numbers, and symbols that Django generates>"

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = ["backend"]
```

Double check now: have those lines of code disappeared from `base.py`? Good. We are slightly less hackable. At the top of the file, add the line `from hello_world.settings.base import *`. What the `*` import from `base` does is make all of those settings that are already defined in our base available in `development` as well, where we're free to overwrite or extend them as necessary.

Since we're embedding our settings files a little deeper in the project by moving them into a `settings` subdirectory, we'll also need to update `BASE_DIR` in `base.py` to point to the correct directory, which is now one level higher (relatively speaking). You can wrap the value in one more `os.path.dirname` call, but I find the following a little easier to read:

```python
BASE_DIR = os.path.abspath(os.path.join(os.path.dirname(__file__), "../../"))
```

Django determines which module to use when running the app with the environment variable `DJANGO_SETTINGS_MODULE`, which should be the module path to the settings that we want to use. To avoid errors, we update the default in `backend/hello_world/wsgi.py` to `'hello_world.settings.base'`, and add the following to our `backend` service in `docker-compose.yml`:

```python
environment:
  - DJANGO_SETTINGS_MODULE=hello_world.settings.development
```

### Add production settings with WhiteNoise

The reason we want to use [WhiteNoise](http://whitenoise.evans.io/en/stable/) in production instead of whatever Django does out-of-the-box is because, by default, Django is very slow to serve frontend assets, whereas WhiteNoise is reasonably fast. Not as fast as professional-grade CDN-AWS-S3-bucket-thingy fast, but fast enough for our purposes.

To start, we need to install WhiteNoise by adding `whitenoise` to `requirements.txt`.

Next, since we have dev-specific settings, let's create `production.py` with settings of its very own. To start, we'll just add production variations of the development settings that we have, which should look something like this:

```python
import os
from hello_world.settings.base import *

SECRET_KEY = os.environ.get("SECRET_KEY")
DEBUG = False
ALLOWED_HOSTS = [os.environ.get("PRODUCTION_HOST")]
```

We'll add the allowed host once we set up an app on Heroku. Note that you can hard-code the allowed host in the settings file, but using an environment variable is a little easier to change if you deploy to a different environment. The `SECRET_KEY` can be any string you want, but for security reasons it should be some long string of random characters (I just use a password generator for mine), and you should save it as an environment/config variable hidden away from the cruel, thieving world. Do not check it into source control!.

To enable WhiteNoise to serve our frontend assets, we add the following to `production.py`:

```python
INSTALLED_APPS.extend(["whitenoise.runserver_nostatic"])

# Must insert after SecurityMiddleware, which is first in settings/common.py
MIDDLEWARE.insert(1, "whitenoise.middleware.WhiteNoiseMiddleware")

TEMPLATES[0]["DIRS"] = [os.path.join(BASE_DIR, "../", "frontend", "build")]

STATICFILES_DIRS = [os.path.join(BASE_DIR, "../", "frontend", "build", "static")]
STATICFILES_STORAGE = "whitenoise.storage.CompressedManifestStaticFilesStorage"
STATIC_ROOT = os.path.join(BASE_DIR, "staticfiles")

STATIC_URL = "/static/"
WHITENOISE_ROOT = os.path.join(BASE_DIR, "../", "frontend", "build", "root")
```

Most of the above comes from the [WhiteNoise documentation](http://whitenoise.evans.io/en/stable/django.html) for implementation in Django, along with a little trial and error to figure out which file paths to use for finding the assets built by React (more on that below). The confusing bit is all the variables that refer to slightly different frontend-asset-related directories.

- `TEMPLATES`: directories with templates (e.g. Jinja) or html files
- `STATICFILES_DIRS`: directory where Django can find html, js, css, and other static assets
- `STATIC_ROOT`: directory to which Django will move those static assets and from which it will serve them when the app is running
- `WHITENOISE_ROOT`: directory where WhiteNoise can find all **non-html** static assets

### Add home URL for production

In addition to changing the settings, we have to make Django aware of the path `/`, because right now it only knows about `/admin` and `/char_count`. So, we'll have to update `/backend/hello_world/urls.py` to look like the following:

```python
from django.contrib import admin
from django.urls import path, re_path
from django.views.generic import TemplateView
from char_count.views import char_count

urlpatterns = [
    path("admin/", admin.site.urls),
    path("char_count", char_count, name="char_count"),
    re_path(".*", TemplateView.as_view(template_name="index.html")),
]
```

Note that we've added a regex path (`.*`) that says to Django, "Any request that you don't have explicit instructions for, just respond by sending them `index.html`". How this works in practice is that in a dev environment, React's Webpack server will still handle calls to `/` (and any path other than the two defined above), but in production, when there's no Webpack server, Django will just shrug its shoulders and serve `index.html` from the static files directory (as defined in the settings above), which is exactly what we want. The reason we use `.*` instead of a specific path is it allows us the freedom to define as many paths as we want for the frontend to handle (with React Router for example) without having to update Django's URLs list.

None of these changes should change our app's functionality on local, so try running `docker-compose up` to make sure nothing breaks.

## Create a production Dockerfile

In order for WhiteNoise to be able to serve our frontend assets, we'll need to include them in the same image as our Django app. There are a few ways we could accomplish this, but I think the simplest is to copy the Dockerfile that builds our backend image and add the installation of our frontend dependencies, along with the building of our assets, to it. Since this image will contain a single app that encompasses both frontend and backend, put it in the project root.

```docker
FROM python:3.6

# Install curl, node, & yarn
RUN apt-get -y install curl \
  && curl -sL https://deb.nodesource.com/setup_8.x | bash \
  && apt-get install nodejs \
  && curl -o- -L https://yarnpkg.com/install.sh | bash

WORKDIR /app/backend

# Install Python dependencies
COPY ./backend/requirements.txt /app/backend/
RUN pip3 install --upgrade pip -r requirements.txt

# Install JS dependencies
WORKDIR /app/frontend

COPY ./frontend/package.json ./frontend/yarn.lock /app/frontend/
RUN $HOME/.yarn/bin/yarn install

# Add the rest of the code
COPY . /app/

# Build static files
RUN $HOME/.yarn/bin/yarn build

# Have to move all static files other than index.html to root/
# for whitenoise middleware
WORKDIR /app/frontend/build

RUN mkdir root && mv *.ico *.js *.json root

# Collect static files
RUN mkdir /app/backend/staticfiles

WORKDIR /app

# SECRET_KEY is only included here to avoid raising an error when generating static files.
# Be sure to add a real SECRET_KEY config variable in Heroku.
RUN DJANGO_SETTINGS_MODULE=hello_world.settings.production \
  SECRET_KEY=somethingsupersecret \
  python3 backend/manage.py collectstatic --noinput

EXPOSE $PORT

CMD python3 backend/manage.py runserver 0.0.0.0:$PORT
```

The Dockerfile above installs everything we need to run both Django and React apps, then builds the frontend assets, then collects those assets for WhiteNoise to serve them. Since the `collectstatic` command makes changes to the files, we want to run it during our build step rather than as a separate command that we run during deployment. You could probably do the latter under some circumstances, but I ran into problems when deploying to Heroku, because they discard post-deployment file changes on free-tier dynos.

Also, note the command that moves static files from `/app/frontend/build` to `/app/frontend/build/root`, leaving `index.html` in place. WhiteNoise needs everything that isn't an HTML file in a separate subdirectory. Otherwise, it gets confused about which files are HTML and which aren't, and nothing ends up getting loaded. Many Bothans died to bring us this information.

## Create an app on Heroku

If you're new to Heroku, their [getting started guide](https://devcenter.heroku.com/articles/getting-started-with-python) will walk you through the basics of creating a generic, non-dockerized Python app. If you don’t have it yet, install the [Heroku CLI](https://devcenter.heroku.com/articles/getting-started-with-python#set-up). We can create a Heroku app by running `heroku create` within our project. Once you've created your new Heroku app, copy the URL displayed by the command, and add it to `ALLOWED_HOSTS` in `settings.production`. Just like adding `backend` to our allowed hosts on dev, we need this to make sure Django's willing to respond to our HTTP requests. (I can't even begin to count the number of blank screens I've repeatedly refreshed with a mix of confusion and despair due to forgetting to add the hostname to `ALLOWED_HOSTS` when deploying to a new environment). If you want to keep it secret, or want greater flexibility, you can add `os.environ.get("PRODUCTION_HOST")` to the allowed hosts instead, then add your Heroku app's URL to its config variables. I'm not sure how strict it is for which URL elements to include or omit, but `<your app name>.herokuapp.com` definitely works.

For environment variables in production, we can use the Heroku CLI to set secure config variables that will be hidden from the public. Heroku has a way of adding these variables with `heroku.yml`, but I always have trouble getting it to work, so I opt for the manual way in this case. This has the added benefit of not having to worry about which variables are okay to commit to source control and which we need to keep secret. To set the config variables, run the following in the terminal:

```bash
heroku config:set PRODUCTION_HOST=<your app name>.herokuapp.com SECRET_KEY=<your secret key> DJANGO_SETTINGS_MODULE=hello_world.settings.production
```

As stated earlier, `PRODUCTION_HOST` is optional (depending on whether you added the app URL to `ALLOWED_HOSTS` directly). `DJANGO_SETTINGS_MODULE` will make sure that the app uses our production settings when running on Heroku.

## Deploy to Heroku

There are a couple of different ways we can deploy Dockerized apps to Heroku, but I like `heroku.yml`, because, like `docker-compose.yml`, it has all the app configs and commands in one place. Heroku has a [good introduction](https://devcenter.heroku.com/articles/build-docker-images-heroku-yml) to how it all works, but for our purposes, we only need the following:

```yaml
build:
  docker:
    web: Dockerfile
run:
  web: python3 backend/manage.py runserver 0.0.0.0:$PORT
```

We also need to run `heroku stack:set container` in the terminal to tell our Heroku app to use Docker rather than one of Heroku's language-specific build packs. Now, deploying is as easy as running `git push heroku master` (if you're on the `master` branch; otherwise, run `git push heroku <your branch>:master`).

Once Heroku is finished building our image and deploying, we can open a browser to `<your app name>.herokuapp.com` and count characters on the CLOOOOOUUUUUD!!!

## Summary

Conceptually, putting the frontend and backend together into a single app that we can deploy to Heroku is very simple, but there are so many little gotchas in the configurations and file structure (not to mention the lack of meaningful error messages whenever one makes a mistake) that I found it devilishly difficult to get it all working. Even going through this process a second time while writing this tutorial, I forgot something here, added the wrong thing there, and spent hours trying to remember how I got it working the first time around, and what terrible sin I might have committed to cause the coding gods to punish me now.

But here we are, having just accomplished the following:

- Configure environment-specific settings for Django.
- Set up WhiteNoise to serve static assets in production.
- Create a production Dockerfile that includes frontend and backend code and dependencies.
- Create a Heroku app and deploy our code to it using `heroku.yml` and the container stack.
