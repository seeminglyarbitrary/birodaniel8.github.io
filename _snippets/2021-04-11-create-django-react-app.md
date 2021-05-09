---
layout: snippet
title: "Create and setup a new Django-React app"
author: "Daniel Biro"
categories: snippet
date: 2021-04-11
---

Create and setup a base for a new Django-React app with the following steps:
### Backend side (Django):

1. New folder for the whole project, then `cd` into it
2. Run: `pipenv install django djangorestframework`
3. **Create Django project**: `django-admin startproject <project_name>`
4. cd into the newly created `<project_name>` folder
5. Change the Python interpreter in VSCode and also in the terminal (open up a bash terminal, it will automatically activate the venv)
6. **Create** an `api` and `frontend` **Django app**: `django-admin startapp <app_name>`
7. Add the new projects to the `settings.py` file eg.: `api.apps.ApiConfig`
8. Also add the `rest_framework` to the apps for React
9. Make migrations (every time there is an update in the database structure): `python manage.py makemigrations`
10. **Migrate**: `python manage.py migrate`
11. **Run the Django server**: `python manage.py runserver`

### Frontend side (React):

1. cd into the **frontend** folder and perform all these steps inside this folder
2. Create a `templates` folder and within a `frontend` folder
3. Create a `static` folder and within `frontend`, `css` and `images` folders
4. Create an `src` folder and within a `components` folder
5. Run (in frontend): `npm init -y`
6. **Install packages**:
    - `npm i webpack webpack-cli @babel/core babel-loader @babel/preset-env @babel/preset-react react react-dom --save-dev`
    - `npm i @material-ui/core @babel/plugin-proposal-class-properties react-router-dom @material-ui/icons`
7. Create the `babel.config.json` file and paste the content from here
8. Create the `webpack.config.js` file and paste the content from here
9. Add these 2 scripts to the `package.json` file and delete the `test` script:
    - `"dev": "webpack --mode development --watch",`
    - `"build": "webpack --mode production"`
10. Create an `index.js` file inside the `src` folder as paste:
{% highlight javascript %}
import App from './components/App'
{% endhighlight %}
11. Create the `index.html` file inside the `templates/frontend` folder and paste the content from here
12. Create a `components` folder inside `src` and first create an `App.js` and then paste the content from here
13. In `frontend/views.py` create an index view: 

{% highlight python %}
def index(request, *args, **kwargs):
    return render(request, "frontend/index.html")
{% endhighlight %}

{:start="14"}
14. In `frontend` create a new `urls.py` file and add the default path directing to the index view:

{% highlight python %}
from django.urls import path
from .views import index
urlpatterns = [
    path("", index),
]
{% endhighlight %}

{:start="15"}
15. In the main `urls.py` add the default path to include the frontend's urls:

{% highlight python %}
path("", include("frontend.urls"))
{% endhighlight %}

{:start="16"}
16. Run: `npm run dev`

### Notes:
 - the Django and npm server both has to run simultaneously
 - whenever an app is created, it has to be added inside the settings.py file
 - if there is a change in the database structure, we have to make migrations
 - this folder/component/file structure for the frontend app is not mandatory and the copy-pasted material only serves as a starting point
 - webpack is a tool to pack/bundle js codes
 - babel takes care of the backward compatibility
 - material-ui provides the UI components (bootstrap is also a common alternative)
 - we have templates/frondend/index.html as the only html file and inside src/ we have the index.js main file
 - it is quite common not to work with the index.js file but to create an app.js component (do everything there) and render it into the index.js file

**Links**:
- [Full Stack Web App With Python & JavaScript from Tech With Tim](https://www.youtube.com/watch?v=JD-age0BPVo&list=PLzMcBGfZo4-kCLWnGmK0jUBmGLaJxvi4j){:target="_blank"}
- [GitHub repo of the App presented by Tech With Tim](https://github.com/techwithtim/Music-Controller-Web-App-Tutorial){:target="_blank"}