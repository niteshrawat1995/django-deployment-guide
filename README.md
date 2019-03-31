# django-deployment-guide
A basic guide on how to deploy django projects in various ways.

This guide will help anyone who want to deploy his/her django application via PaaS(like Heroku) or IaaS(like AWS EC2, DigitalOcean droplets etc..)

Different ways are given below:

##1. Deploying on Heroku:

We can choose to use Heroku for several reasons:

-> Heroku has a free tier that is really free (albeit with some limitations).

-> As a PaaS, Heroku takes care of a lot of the web infrastructure for us. This makes it much easier to get started, because you don't worry about servers, load balancers, reverse proxies, or any of the other web infrastructure that Heroku provides for us under the hood.

->While it does have some limitations these will not affect this particular application. For example:
    -> Heroku provides only short-lived storage so user-uploaded files cannot safely be stored on Heroku itself.
    -> The free tier will sleep an inactive web app if there are no requests within a half hour period. The site may then take several seconds to respond when it is woken up.
    -> The free tier limits the time that your site is running to a certain amount of hours every month (not including the time that the site is "asleep"). This is fine for a low use/demonstration site, but will not be suitable if 100% uptime is required.
Other limitations are listed in Limits (Heroku docs).
-> Mostly it just works, and if you end up loving it, scaling your app is very easy.

# NOTE/ prerequisites: 
    1. Heroku doesnt support filebased storage to store media files like images and other user uplaods.Thus, we can't use the default SQLite database on Heroku because it is file-based, and it would be deleted from the ephemeral file system every time the application restarts (typically once a day, and every time the application or its configuration variables are changed).
    For this we are going to use AWS S3 service to store our media files. Please refer How to move project file storage to Amazon S3.

    2. In order to execute your application Heroku needs to be able to set up the appropriate environment and dependencies, and also understand how it is launched.For Django apps we provide this information in a number of text files, in this guide we will use Procfile method.
    Procfile: A list of processes to be executed to start the web application. For Django this will usually be the Gunicorn web application server (with a .wsgi script).
    3. Heroku does its deployment with git version control, so make sure you use it.
    4. STATIC_ROOT = os.path.join(BASE_DIR, 'static') and MEDIA_ROOT = os.path.join(BASE_DIR, 'media') are set in settings.py.
    5. ALLOWED_HOST = ["*"] and DEBUG = FALSE in settings.py are set.
    6. Sensitive informations like SECRET_KEY, Database credentials, AWS credentials must come from environment variable and not from source code.

STEPS:
1. Create account on Heroku: https://signup.heroku.com.

2. Install heroku cli on your system via 
$sudo snap install --classic heroku(ubuntu), $brew tap heroku/brew && brew install heroku.(mac)

3. Activate your virtual env  $sudo source env/bin/activate or create one via 
$sudo apt install virtualenv && sudo virtualenv env && sudo source env/bin/activate

4. Login to heroku via terminal
$ heroku login 
and navigate to your project directory

5. Install gunicorn:
$ pip install gunicorn
$ pip freeze > requirements.txt

6. Create app on heroku and open the url:
$ heroku create <your_project_name>
$ heroku open

7. Push code to Heroku remote
$ git push heroku master

8. Create Procfile in the project directory:
$ touch Procfile
$ sudo nano Procfile
paste the below line there:
web: gunicorn <your_project_name>.wsgi
$ git add -A
$ git commit -m "Created Procfile file"
$ git push heroku master

9. Set Environment variables like SECRET_KEY, Database credentials, AWS credentials for Heroku using heroku config:set <VAR_NAME>=<VAR_VALUE>
example:
$ heroku config:set SECRET_KEY="asdsabsakjfksajkfas"

10. Push the code to heroku remote master,  refer step 8.
11. Check if heroku provide a postgres DB.
$ heroku addons
$ heroku pg (check db details)
12.iF no DB is present then create a new database on Heroku (we didnt want to use sqlite3 on prod)
$ heroku addons: create heroku-postgresql:hobby-dev (free tier)
13. Install django-heroku
$ pip install django-heroku
This package will automatically configure our database url and will also take care of connecting our static assessts to gunicorn using a package called 'WhiteNoise'
Open settings.py
import os
import django_heroku
# At the bottom
django_heroku.settings(locals())
Now, again push the changes to heroku remote master, refer step 8.
14. Migrate the tables and create superuser. Go inside your machine(dyno) via
$ heroku run bash
$ python manage.py migrate
$ python manage.py createsuperuser

15. DONE, you can now visit your site on the url provided by heroku.

*** You can also version control via heroku:
$ heroku releases




