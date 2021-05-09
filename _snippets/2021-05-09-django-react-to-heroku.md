---
layout: snippet
title: "Django & React App deployment to Heroku"
author: "Daniel Biro"
categories: snippet
date: 2021-05-09
---

Deploying my Django & React app to Heroku on Windows:
 - Many steps must be followed from the [Django only deployment](https://seeminglyarbitrary.github.io/snippets/2021-04-03-django-to-heroku.html)
 - Some noteable difference:
    - Recently I have decided to keep the pipfile and package.json related files outside the main project folder
    - The main project folder usually contains an app for the backend/api (Django) and another for the frontend (React)
    - This means the Procfile we create will be in a different folder than the manage.py file

New steps:
1. Create a new heroku app **outside** the project's folder
2. Add the Python and Node.js buildpacks:
    1. run `heroku buildpacks:set heroku/python`
    2. run `heroku buildpacks:add --index 1 heroku/nodejs`
3. Create a new `Procfile` and paste `web: gunicorn --pythonpath <project_name> <project_name>.wsgi --log-file -` into it (this is now different because we have the manage.py file in a different location)
4. Create another file named `Procfile_windows` and paste `web: py <project_name>/manage.py runserver 127.0.0.1:8000` into it and test it
5. Add to the `package.json` file:
{% highlight python %}
"scripts": {
    ...
    "postinstall": "npm run build"
},
"engines": {
    "npm": "6.14.11",
    "node": "14.16.0"
}, 
{% endhighlight %}
    
6. Setup PostGres
7. Commit and push the changes and deploy by connecting GitHub to Heroku

(Some extra note: `pipenv shell` can create the Python environment from an existing pipfile, `npm i` creates the node_modules folder and its contents)

**Links**:
- [Deploy your React-Django app on Heroku](https://alphacoder.xyz/deploy-react-django-app-on-heroku/)
- [How can I modify Procfile to run Gunicorn process in a non-standard folder on Heroku?](https://stackoverflow.com/questions/16416172/how-can-i-modify-procfile-to-run-gunicorn-process-in-a-non-standard-folder-on-he)