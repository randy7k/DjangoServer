<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Thanks again! Now go create something AMAZING! :D
-->



<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]
-->


<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">Django server</h3>
  <p align="center">
    Setting up Django with Nginx, Gunicorn, virtualenv, supervisor and PostgreSQL
  </p> 
</p>



<!-- TABLE OF CONTENTS -->
<!-- <details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>

 -->

<!-- ABOUT THE PROJECT -->
## About 

<!-- [![Product Name Screen Shot][product-screenshot]](https://example.com) -->

Django is an efficient, versatile and dynamically evolving web application development framework. When Django initially gained popularity, the recommended setup for running Django applications was based around Apache with mod_wsgi. The art of running Django advanced and these days the recommended configuration is more efficient and resilient, but also more complex and includes such tools as: Nginx, Gunicorn, virtualenv, supervisord and PostgreSQL.

In this text I will explain how to combine all of these components into a Django server running on Linux.

### Built With

In this text I will explain how to combine all of these components into a Django server running on Linux.

* [Django]()
* [Nginx]()
* [Gunicorn]()
* [virtualenv]()
* [supervisor]()
* [PostgreSQL]()

<!-- GETTING STARTED -->
## Getting Started

I assume you have a server available on which you have root privileges.

I’m also assuming you configured your DNS to point a domain at the server’s IP. In this text, I pretend your domain is example.com

### Prerequisites

Update your system
Let’s get started by making sure our system is up to date.


  ```sh
  sudo apt update
  sudo apt upgrade
  ```
  
### PostgreSQL
  To install PostgreSQL system run this command:

  ```sh
  sudo apt install postgresql postgresql-contrib
  ```
  Create a database user and a new database for the app. 
  
```sh
  sudo su - postgres
  postgres@django:~$ createuser --interactive -P
  Enter name of role to add: hello_django
  Enter password for new role: 
  Enter it again: 
  Shall the new role be a superuser? (y/n) n
  Shall the new role be allowed to create databases? (y/n) n
  Shall the new role be allowed to create more new roles? (y/n) n
  postgres@django:~$

  postgres@django:~$ createdb --owner hello_django hello
  postgres@django:~$ logout
  $
  ```
### Application user

Even though Django has a pretty good security track record, web applications can become compromised. If the application has limited access to resources on your server, potential damage can also be limited. Your web applications should run as system users with limited privileges.

Create a user for your app, named hello and assigned to a system group called 'webapps' .

  ```sh
  sudo groupadd --system webapps
  sudo useradd --system --gid webapps --shell /bin/bash --home /webapps/hello_django hello
  ```
### Install virtualenv and create an environment for you app

Virtualenv is a tool which allows you to create separate Python environments on your system. This allows you to run applications with different sets of requirements concurrently (e.g. one based on Django 1.5, another based on 1.6). virtualenv is easy to install on Debian:

  ```sh
  sudo apt-get install python3-venv
  ```
  
### Create and activate an environment for your application

I like to keep all my web apps in the /webapps/ directory. If you prefer /var/www/, /srv/ or something else, use that instead. Create a directory to store your application in /webapps/hello_django/ and change the owner of that directory to your application user hello


  ```sh
  sudo mkdir -p /webapps/hello_django/
  sudo chown hello /webapps/hello_django/
  ```
As the application user create a virtual Python environment in the application directory:

  ```sh
  sudo su - hello
  hello@django:~$ cd /webapps/hello_django/
  hello@django:~$ virtualenv .

  New python executable in hello_django/bin/python
  Installing distribute..............done.
  Installing pip.....................done.

  hello@django:~$ source bin/activate
  (hello_django)hello@django:~$ 
  ```
  
  Your environment is now activated and you can proceed to install Django inside it.

  ```sh
  (hello_django)hello@django:~$ pip install django

  Downloading/unpacking django
  (...)
  Installing collected packages: django
  (...)
  Successfully installed django
  Cleaning up...
  ```
Your environment with Django should be ready to use. Go ahead and create an empty Django project.


  ```sh
  (hello_django)hello@django:~$ django-admin.py startproject hello
  ```
You can test it by running the development server:

  ```sh
  (hello_django)hello@django:~$ cd hello
  (hello_django)hello@django:~$ python manage.py runserver example.com:8000
  Validating models...

  0 errors found
  June 09, 2013 - 06:12:00
  Django version 1.5.1, using settings 'hello.settings'
  Development server is running at http://example.com:8000/
  Quit the server with CONTROL-C.
  ```
You should now be able to access your development server from http://example.com:8000

Allowing other users write access to the application directory
Your application will run as the user hello, who owns the entire application directory. If you want regular user to be able to change application files, you can set the group owner of the directory to users and give the group write permissions.

  ```sh
  $ sudo chown -R hello:users /webapps/hello_django
  $ sudo chmod -R g+w /webapps/hello_django
  ```

You can check what groups you’re a member of by issuing the groups command or id.
  ```sh
  $ id
  uid=1000(michal) gid=1000(michal) groups=1000(michal),27(sudo),100(users)
  ```
If you’re not a member of users, you can add yourself to the group with this command:

  ```sh
$ sudo usermod -a -G users `whoami`
  ```
### Configure PostgreSQL to work with Django

In order to use Django with PostgreSQL you will need to install the psycopg2 database adapter in your virtual environment. This step requires the compilation of a native extension (written in C). The compilation will fail if it cannot find header files and static libraries required for linking C programs with libpq (library for communication with Postgres) and building Python modules (python-dev package). We have to install these two packages first, then we can install psycopg2 using PIP.

Install dependencies:

  ```sh
  $ sudo aptitude install libpq-dev python-dev
  ```

Install psycopg2 database adapter:

  ```sh
  (hello_django)hello@django:~$ pip install psycopg2
  ```
You can now configure the databases section in your settings.py:

  ```
  DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'hello',
        'USER': 'hello_django',
        'PASSWORD': '1Ak5RTQt7mtw0OREsfPhJYzXIak41gnrm5NWYEosCeIduJck10awIzoys1wvbL8',
        'HOST': 'localhost',
        'PORT': '',                      # Set to empty string for default.
      }
  }
  ```
And finally build the initial database for Django:

  ```sh
  (hello_django)hello@django:~$ python manage.py migrate
  ```

### Gunicorn

In production we won’t be using Django’s single-threaded development server, but a dedicated application server called gunicorn.

Install gunicorn in your application’s virtual environment:

  ```sh
  (hello_django)hello@django:~$ pip install gunicorn
  Downloading/unpacking gunicorn
    Downloading gunicorn-0.17.4.tar.gz (372Kb): 372Kb downloaded
    Running setup.py egg_info for package gunicorn

  Installing collected packages: gunicorn
    Running setup.py install for gunicorn

      Installing gunicorn_paster script to /webapps/hello_django/bin
      Installing gunicorn script to /webapps/hello_django/bin
      Installing gunicorn_django script to /webapps/hello_django/bin
  Successfully installed gunicorn
  Cleaning up...
  ```

Now that you have gunicorn, you can test whether it can serve your Django application by running the following command:

  ```sh
  (hello_django)hello@django:~$ gunicorn hello.wsgi:application --bind example.com:8001
  ```
You should now be able to access the Gunicorn server from http://example.com:8001 . I intentionally changed port 8000 to 8001 to force your browser to establish a new connection.

Gunicorn is installed and ready to serve your app. Let’s set some configuration options to make it more useful. I like to set a number of parameters, so let’s put them all into a small BASH script, which I save as bin/gunicorn_start.bash

  ```sh
  #!/bin/bash

NAME="hello_app"                                  # Name of the application
DJANGODIR=/webapps/hello_django/hello             # Django project directory
SOCKFILE=/webapps/hello_django/run/gunicorn.sock  # we will communicte using this unix socket
USER=hello                                        # the user to run as
GROUP=webapps                                     # the group to run as
NUM_WORKERS=3                                     # how many worker processes should Gunicorn spawn
DJANGO_SETTINGS_MODULE=hello.settings             # which settings file should Django use
DJANGO_WSGI_MODULE=hello.wsgi                     # WSGI module name

echo "Starting $NAME as `whoami`"

# Activate the virtual environment
cd $DJANGODIR
source ../bin/activate
export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DJANGODIR:$PYTHONPATH

# Create the run directory if it doesn't exist
RUNDIR=$(dirname $SOCKFILE)
test -d $RUNDIR || mkdir -p $RUNDIR

# Start your Django Unicorn
# Programs meant to be run under supervisor should not daemonize themselves (do not use --daemon)
exec ../bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $NUM_WORKERS \
  --user=$USER --group=$GROUP \
  --bind=unix:$SOCKFILE \
  --log-level=debug \
  --log-file=-
  ```
  
Set the executable bit on the gunicorn_start.bash script:
  
  ```sh
  $ sudo chmod u+x bin/gunicorn_start.bash
  ```
  You can test your gunicorn_start.bash script by running it as the user hello.
 
  ```sh
$ sudo su - hello
hello@django:~$ bin/gunicorn_start.bash
Starting hello_app as hello
2013-06-09 14:21:45 [10724] [INFO] Starting gunicorn 18.0
2013-06-09 14:21:45 [10724] [DEBUG] Arbiter booted
2013-06-09 14:21:45 [10724] [INFO] Listening at: unix:/webapps/hello_django/run/gunicorn.sock (10724)
2013-06-09 14:21:45 [10724] [INFO] Using worker: sync
2013-06-09 14:21:45 [10735] [INFO] Booting worker with pid: 10735
2013-06-09 14:21:45 [10736] [INFO] Booting worker with pid: 10736
2013-06-09 14:21:45 [10737] [INFO] Booting worker with pid: 10737

^C (CONTROL-C to kill Gunicorn)

2013-06-09 14:21:48 [10736] [INFO] Worker exiting (pid: 10736)
2013-06-09 14:21:48 [10735] [INFO] Worker exiting (pid: 10735)
2013-06-09 14:21:48 [10724] [INFO] Handling signal: int
2013-06-09 14:21:48 [10737] [INFO] Worker exiting (pid: 10737)
2013-06-09 14:21:48 [10724] [INFO] Shutting down: Master
$ exit
  ```
Note the parameters set in gunicorn_start.bash You’ll need to set the paths and filenames to match your setup.

As a rule-of-thumb set the --workers (NUM_WORKERS) according to the following formula: 2 * CPUs + 1. The idea being, that at any given time half of your workers will be busy doing I/O. For a single CPU machine it would give you 3.

The --name (NAME) argument specifies how your application will identify itself in programs such as top or ps. It defaults to gunicorn, which might make it harder to distinguish from other apps if you have multiple Gunicorn-powered applications running on the same server.

In order for the --name argument to have an effect you need to install a Python module called setproctitle. To build this native extension pip needs to have access to C header files for Python. You can add them to your system with the python-dev package and then install setproctitle.
  
  ```sh
  $ sudo aptitude install python-dev
  (hello_django)hello@django:~$ pip install setproctitle
  ```
 
Now when you list processes, you should see which gunicorn belongs to which application.

  ```sh
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
(...)
hello    11588  0.7  0.2  58400 11568 ?        S    14:52   0:00 gunicorn: master [hello_app]
hello    11602  0.5  0.3  66584 16040 ?        S    14:52   0:00 gunicorn: worker [hello_app]
hello    11603  0.5  0.3  66592 16044 ?        S    14:52   0:00 gunicorn: worker [hello_app]
hello    11604  0.5  0.3  66604 16052 ?        S    14:52   0:00 gunicorn: worker [hello_app]
  ```
### Starting and monitoring with Supervisor

Your gunicorn_start script should now be ready and working. We need to make sure that it starts automatically with the system and that it can automatically restart if for some reason it exits unexpectedly. These tasks can easily be handled by a service called supervisord. Installation is simple:

  ```sh
  $ sudo aptitude install supervisor
  ```
 When Supervisor is installed you can give it programs to start and watch by creating configuration files in the /etc/supervisor/conf.d directory. For our hello application we’ll create a file named /etc/supervisor/conf.d/hello.conf with this content:

hello.conf
  ```sh
[program:hello]
command = /webapps/hello_django/bin/gunicorn_start.bash               ; Command to start app
user = hello                                                          ; User to run as
stdout_logfile = /webapps/hello_django/logs/gunicorn_supervisor.log   ; Where to write log messages
redirect_stderr = true                                                ; Save stderr in the same log
environment=LANG=en_US.UTF-8,LC_ALL=en_US.UTF-8                       ; Set UTF-8 as default encoding
view rawhello.conf
  ```
 
You can set many other options, but this basic configuration should suffice.

Create the file to store your application’s log messages:

  ```sh
hello@django:~$ mkdir -p /webapps/hello_django/logs/
hello@django:~$ touch /webapps/hello_django/logs/gunicorn_supervisor.log
  ```
After you save the configuration file for your program you can ask supervisor to reread configuration files and update (which will start your the newly registered app).

```sh
$ sudo supervisorctl reread
hello: available
$ sudo supervisorctl update
hello: added process group
  ```
 You can also check the status of your app or start, stop or restart it using supervisor.
  
  ```sh
$ sudo supervisorctl status hello                       
hello                            RUNNING    pid 18020, uptime 0:00:50
$ sudo supervisorctl stop hello  
hello: stopped
$ sudo supervisorctl start hello                        
hello: started
$ sudo supervisorctl restart hello 
hello: stopped
hello: started
  ```
Your application should now be automatically started after a system reboot and automatically restarted if it ever crashed for some reason.

### Nginx

Time to set up Nginx as a server for out application and its static files. Install and start Nginx:


  ```sh
  $ sudo aptitude install nginx
  $ sudo service nginx start
  ```
 
You can navigate to your server (http://example.com) with your browser and Nginx should greet you with the words “Welcome to nginx!”.

Create an Nginx virtual server configuration for Django
Each Nginx virtual server should be described by a file in the /etc/nginx/sites-available directory. You select which sites you want to enable by making symbolic links to those in the /etc/nginx/sites-enabled directory.

Create a new nginx server configuration file for your Django application running on example.com in /etc/nginx/sites-available/hello. The file should contain something along the following lines. A more detailed example is available from the folks who make Gunicorn.

hello.nginxconf
  ```sh
upstream hello_app_server {
  # fail_timeout=0 means we always retry an upstream even if it failed
  # to return a good HTTP response (in case the Unicorn master nukes a
  # single worker for timing out).

  server unix:/webapps/hello_django/run/gunicorn.sock fail_timeout=0;
}

server {

    listen   80;
    server_name example.com;

    client_max_body_size 4G;

    access_log /webapps/hello_django/logs/nginx-access.log;
    error_log /webapps/hello_django/logs/nginx-error.log;
 
    location /static/ {
        alias   /webapps/hello_django/static/;
    }
    
    location /media/ {
        alias   /webapps/hello_django/media/;
    }

    location / {
        # an HTTP header important enough to have its own Wikipedia entry:
        #   http://en.wikipedia.org/wiki/X-Forwarded-For
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # enable this if and only if you use HTTPS, this helps Rack
        # set the proper protocol for doing redirects:
        # proxy_set_header X-Forwarded-Proto https;

        # pass the Host: header from the client right along so redirects
        # can be set properly within the Rack application
        proxy_set_header Host $http_host;

        # we don't want nginx trying to do something clever with
        # redirects, we set the Host: header above already.
        proxy_redirect off;

        # set "proxy_buffering off" *only* for Rainbows! when doing
        # Comet/long-poll stuff.  It's also safe to set if you're
        # using only serving fast clients with Unicorn + nginx.
        # Otherwise you _want_ nginx to buffer responses to slow
        # clients, really.
        # proxy_buffering off;

        # Try to serve static files from nginx, no point in making an
        # *application* server like Unicorn/Rainbows! serve static files.
        if (!-f $request_filename) {
            proxy_pass http://hello_app_server;
            break;
        }
    }

    # Error pages
    error_page 500 502 503 504 /500.html;
    location = /500.html {
        root /webapps/hello_django/static/;
    }
}
  ```
Create a symbolic link in the sites-enabled folder:

  ```sh
$ sudo ln -s /etc/nginx/sites-available/hello /etc/nginx/sites-enabled/hello
  ```

Restart Nginx:

  ```sh
$ sudo service nginx restart 
  ```
If you navigate to your site, you should now see your Django welcome-page powered by Nginx and Gunicorn. Go ahead and develop to your heart’s content.

At this stage you may find that instead of the Django welcome-page, you encounter the default “Welcome to nginx!” page. This may be caused by the default configuration file, which is installed with Nginx and masks your new site’s configuration. If you don’t plan to use it, delete the symbolic link to this file from /etc/nginx/sites-enabled.

If you run into any problems with the above setup, please drop me a line.

### Final directory structure
If you followed this tutorial, you should have created a directory structure resembling this:
  
  
  ```
sh/webapps/hello_django/
├── bin                          <= Directory created by virtualenv
│   ├── activate                 <= Environment activation script
│   ├── django-admin.py
│   ├── gunicorn
│   ├── gunicorn_django
│   ├── gunicorn_start           <= Script to start application with Gunicorn
│   └── python
├── hello                        <= Django project directory, add this to PYTHONPATH
│   ├── manage.py
│   ├── project_application_1
│   ├── project_application_2
│   └── hello                    <= Project settings directory
│       ├── __init__.py
│       ├── settings.py          <= hello.settings - settings module Gunicorn will use
│       ├── urls.py
│       └── wsgi.py              <= hello.wsgi - WSGI module Gunicorn will use
├── include
│   └── python2.7 -> /usr/include/python2.7
├── lib
│   └── python2.7
├── lib64 -> /webapps/hello_django/lib
├── logs                         <= Application logs directory
│   ├── gunicorn_supervisor.log
│   ├── nginx-access.log
│   └── nginx-error.log
├── media                        <= User uploaded files folder
├── run
│   └── gunicorn.sock 
└── static                       <= Collect and serve static files from here
  ```
 Uninstalling the Django application
If time comes to remove the application, follow these steps.

Remove the virtual server from Nginx sites-enabled folder:
  ```sh
$ sudo rm /etc/nginx/sites-enabled/hello_django
  ```
Restart Nginx:

  ```sh
$ sudo service nginx restart 
  ```
If you never plan to use this application again, you can remove its config file also from the sites-available directory

  ```sh
$ sudo rm /etc/nginx/sites-available/hello_django
  ```
Stop the application with Supervisor:

  ```sh
$ sudo supervisorctl stop hello
  ```
Remove the application from Supervisor’s control scripts directory:

  ```sh
$ sudo rm /etc/supervisor/conf.d/hello.conf
  ```
If you never plan to use this application again, you can now remove its entire directory from webapps:

  ```sh
$ sudo rm -r /webapps/hello_django
  ```
### Different settings in production
You will probably want to use different settings on your production server and different settings during development.

To achieve this I usually create a second settings file called settings_local.py, which contains overrides of some values, but imports every default like so:
  ```sh
from hello.settings import *
DATABASES = ...
  ```
 You can then tell Django to use this local settings file by specifying the environment variable DJANGO_SETTINGS_MODULE=hello.settings_local.
  
<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements
* [GitHub Emoji Cheat Sheet](https://www.webpagefx.com/tools/emoji-cheat-sheet)
* [Img Shields](https://shields.io)
* [Choose an Open Source License](https://choosealicense.com)
* [GitHub Pages](https://pages.github.com)
* [Animate.css](https://daneden.github.io/animate.css)
* [Loaders.css](https://connoratherton.com/loaders)
* [Slick Carousel](https://kenwheeler.github.io/slick)
* [Smooth Scroll](https://github.com/cferdinandi/smooth-scroll)
* [Sticky Kit](http://leafo.net/sticky-kit)
* [JVectorMap](http://jvectormap.com)
* [Font Awesome](https://fontawesome.com)





<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/othneildrew/Best-README-Template.svg?style=for-the-badge
[contributors-url]: https://github.com/othneildrew/Best-README-Template/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/othneildrew/Best-README-Template.svg?style=for-the-badge
[forks-url]: https://github.com/othneildrew/Best-README-Template/network/members
[stars-shield]: https://img.shields.io/github/stars/othneildrew/Best-README-Template.svg?style=for-the-badge
[stars-url]: https://github.com/othneildrew/Best-README-Template/stargazers
[issues-shield]: https://img.shields.io/github/issues/othneildrew/Best-README-Template.svg?style=for-the-badge
[issues-url]: https://github.com/othneildrew/Best-README-Template/issues
[license-shield]: https://img.shields.io/github/license/othneildrew/Best-README-Template.svg?style=for-the-badge
[license-url]: https://github.com/othneildrew/Best-README-Template/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/othneildrew
[product-screenshot]: images/screenshot.png
