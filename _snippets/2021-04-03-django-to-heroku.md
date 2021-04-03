---
layout: snippet
title: "Django app deployment to Heroku"
author: "Daniel Biro"
categories: snippet
---

Deploying my Django app to Heroku on Windows (assuming the Django app is already functioning and a virtual environment is created by `pipenv`):
1. Register a **Heroku account** and download/install the Heroku CLI
2. **Create a Heroku app**:
    1. Navigate to the app folder (where the manage.py file is)
    2. In the terminal create a new Heroku app: `heroku login`
    3. Initialize a Git repo and do a first commit:
        (`git init`, `git add .`, `git commit -m "first commit"`)
    4. `heroku create <heroku_app_name>`
    5. `heroku git:remote -a <heroku_app_name>`
3. Install packages to pipenv:
    1. **gunicorn**: this will run our app on Heroku
    2. **waitress**: local serve on Windows
    3. **whitenoise**: handle static files
    4. **psycopg2**: PostGres database
3. Local tests:
    1. Open up a terminal and activate the venv (I am using the bash terminal in VSCode)
    2. Run: `waitress-serve --listen=127.0.0.1:8000 <app_name>.wsgi:application`
    3. Make sure it is functioning
4. **Procfile**:
    1. Create file named `Procfile` and paste `web: gunicorn <app_name>.wsgi --log-file -` into it
    2. `heroku local` doesn't work with Windows, but this procfile will be used in Heroku remote
    3. Create another file named `Procfile_windows` and paste `web: py manage.py runserver 127.0.0.1:8000` into it
    4. Run: `heroku local -f Procfile_windows`
    5. Make sure it is functioning
5. We **don't** need any requirements.txt file as we already have the `Pipfile`
6. Additional modifications in settings.py:
    1. Add `STATIC_ROOT = os.path.join(BASE_DIR, "staticfiles")` at the end of the file
    2. Add `['127.0.0.1', '<heroku_app_name>.herokuapp.com']` to the `ALLOWED_HOSTS` list
    3. Add `whitenoise.middleware.WhiteNoiseMiddleware` to the `MIDDLEWARE` list
7. Commit the changes and push to heroku master: `git push heroku master`
8. The app should be up and **running on Heroku**
9. SQLite is working just fine but the filesystem is not permanent on Heroku, so it will delete our database soon, we don’t want that so we will use **PostGres**:
    1. Modify the database in the settings.py: `'ENGINE': 'django.db.backends.postgresql'`
	2. On the Heroku Dashboard add a new add-on: Heroku PostGres
	3. On the settings tab you have the `config vars`: reveal config vars
	4. There is a long string there and we will extract the info from that and att the `NAME`, `HOST`, `PORT`, `USER`, `PASSWORD` keys to the DATABASES/default dictionary in the settings.py file
    5. The string we have is contructed as:
    `postgres://<USER>:<PASSWORD>@<HOST>:<PORT>/<NAME>`
	6. Migrate the database (this will of course be an empty one): `python manage.py migrate`
	7. Commit and push again
	8. We can set `DEBUG` to False at the end


**Links**:
- [Deploy a Django App to Heroku](https://www.youtube.com/watch?v=GMbVzl_aLxM)