---
layout: snippet
title: "Django app deployment to Heroku"
author: "Daniel Biro"
categories: snippet
---

Deploying my Django app to Heroku on Windows (assuming the Django app is already functioning and a virtual environment is created by `pipenv`):
1. Register a **Heroku account** and download/install the Heroku CLI
2. **Create a Heroku app**:
    1. Navigate to the app folder (where the manage.py file is; all the commands below should be executed from this folder)
    2. Create a **private** Git repo on GitHub and follow the instructions on how to push the files to GitHub from this folder:
    3. Login to Heroku: `heroku login`
    4. Create a new app: `heroku create <heroku_app_name>`
3. Install packages to pipenv:
    1. **django**: if you haven't already added
    2. **gunicorn**: this will run our app on Heroku
    3. **waitress**: local serve on Windows
    4. **whitenoise**: handle static files
    5. **psycopg2**: PostGres database
    6. And all the other packages used in the app
4. Local tests:
    1. Open up a terminal and activate the venv (I am using the bash terminal in VSCode)
    2. Run: `waitress-serve --listen=127.0.0.1:8000 <app_name>.wsgi:application`
    3. Make sure it is functioning
5. **Procfile**:
    1. Create file named `Procfile` and paste `web: gunicorn <app_name>.wsgi --log-file -` into it
    2. `heroku local` doesn't work with Windows, but this procfile will be used in Heroku remote
    3. Create another file named `Procfile_windows` and paste `web: py manage.py runserver 127.0.0.1:8000` into it
    4. Run: `heroku local -f Procfile_windows`
    5. Make sure it is functioning
6. We **don't** need any requirements.txt file as we already have the `Pipfile`
7. Additional modifications in settings.py:
    1. Add `STATIC_ROOT = os.path.join(BASE_DIR, "staticfiles")` at the end of the file
    2. Add `['127.0.0.1', '<heroku_app_name>.herokuapp.com']` to the `ALLOWED_HOSTS` list
    3. Add `whitenoise.middleware.WhiteNoiseMiddleware` to the `MIDDLEWARE` list (2nd element)
8. Commit and push the changes
9. Connect a **GitHub repo to Heroku**:
    1. On Heroku Dashboard, navigate to Deploy and select the GitHub deployment method
    2. Select the repo and the branch to be used
    3. Enable automatic deployment
    4. Do a manual deployment now to first deploy the app
10. After the successful deployment, the app should be up and **running on Heroku**
11. The SQLite database is working just fine now but the filesystem is not permanent on Heroku, so it will delete our database soon, we don’t want that so we will use **PostGres**:
    1. Modify the database in the settings.py: `'ENGINE': 'django.db.backends.postgresql'`
	2. On the Heroku Dashboard add a new add-on: Heroku PostGres
	3. On the settings tab you have the `config vars`: reveal config vars
	4. There is a long string there and we will extract the info from that and att the `NAME`, `HOST`, `PORT`, `USER`, `PASSWORD` keys to the DATABASES/default dictionary in the settings.py file
    5. The string we have is contructed as:
    `postgres://<USER>:<PASSWORD>@<HOST>:<PORT>/<NAME>`
	6. Migrate the database (this will of course be an empty one): `python manage.py migrate`
    7. Create a new super user: `python manage.py createsuperuser`
	7. Commit and push again
	8. We can set `DEBUG` to False at the end

The private Git repository is necessary as some of the configurations with the passwords are sitting in the settings.py file.

Alternatively one can push the files directly to Heroku and then there is no need for a GitHub repository. To do this, initialize a git repo in your folder and run `heroku git:remote -a <heroku_app_name>` right after the first commit. After that all the steps are the same except you can skip step (9) and just push to Heroku by `git push heroku master`.

**Links**:
- [Deploy a Django App to Heroku](https://www.youtube.com/watch?v=GMbVzl_aLxM)
- [Django Everywhere-Host Your Django App for Free on Heroku](https://studygyaan.com/django/django-everywhere-host-your-django-app-for-free-on-heroku)
- [Gunicorn doesn't work on Windows'](https://stackoverflow.com/questions/11087682/does-gunicorn-run-on-windows)
- [Pipfile vs requirements.txt](https://stackoverflow.com/questions/63252388/requirements-txt-vs-pipfile-in-heroku-flask-webapp-deployment)