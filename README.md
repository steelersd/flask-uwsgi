Flask with uWSGI + Nginx
===
This tutorial shows you to set up a simple Flask app with uWSGI + Nginx locally (Using localhost or 127.0.0.1).

At the end of this tutorial, you will be able to do the following:

- Start or stop the app using Upstart
- Visit the app at http://flask-uwsgi.com

Setting up the Python environment
---
First, install the following packages.

```bash
sudo apt-get install build-essential python-dev python-pip
```

`build-essential` and `python-dev` are used to compile C extensions. Pip is used to install system commands.

Next, install the following Pip packages.

```bash
sudo pip install virtualenv uwsgi
```

Next, create the Virtualenv for this tutorial. (Can also use https://virtualenvwrapper.readthedocs.org/en/latest/)

```bash
cd flask-uwsgi
virtualenv env
source env/bin/activate
pip install flask
```

Next, run the Flask app using uWSGI. --home will depend on your virtual environment location. --wsgi-file will be the location of your flask_uwsgi.py file. Mine are as follows

```bash
uwsgi --http 127.0.1:8080 --home ~/.virtualenvs/env --wsgi-file ~/code/python/flask-uwsgi/flask-uwsgi/flask_uwsgi.py --callable app --master
```

You should be able to visit http://localhost:8080.

Creating an Upstart service for uWSGI
---
First, set up the directories the service will use.

```bash
# Create a directory for the UNIX sockets
sudo mkdir /var/run/flask-uwsgi
sudo chown www-data:www-data /var/run/flask-uwsgi

# Create a directory for the logs
sudo mkdir /var/log/flask-uwsgi
sudo chown www-data:www-data /var/log/flask-uwsgi

# Create a directory for the configs
sudo mkdir /etc/flask-uwsgi
```

Next, create the init script.

- Copy the init script template to /etc/init/flask-uwsgi.conf.
- Replace /path/to/flask-uwsgi with the path to this directory.
- **Note:** Do not use ~/ in flask-uwsgi.conf. It will not expand.
- Example: Use /home/adam/code/python/flask-uwsgi/flask-uwsgi/ NOT ~/code/python/flask-uwsgi/flask-uwsgi/

Next, create the config file.

- Copy the config template to /etc/flask-uwsgi/flask-uwsgi.ini.
- Set uid and gid to the numeric uid and gid for www-data.

```bash
    # Get uid for user www-data
    id -u www-data
```

Next, start uWSGI.

    sudo service flask-uwsgi start

Next, check that uWSGI is running.

    sudo less /var/log/flask-uwsgi/flask-uwsgi.log

You should see messages from uWSGI.

Connecting uWSGI to Nginx
---
First, install Nginx (if you haven't already).

```bash
sudo apt-get install nginx
```

Next, create the Nginx config.

- Copy the config template to /etc/nginx/sites-available/flask-uwsgi.
- Replace ubuntu.local with your machine's name.

Next, enable the config.

```bash
sudo ln -sf /etc/nginx/sites-available/flask-uwsgi /etc/nginx/sites-enabled
sudo service nginx reload
```

Next, update you /etc/hosts file to use flask-uwsgi.com

127.0.1.1 flask-uwsgi.com

Next, check that the app is reachable through Nginx. You should be able to visit http://flask-uwsgi.com.

Congratulations! You have deployed an app to uWSGI + Nginx.

Code reloading
---
You can reload code (or reload the uWSGI config) by sending the HUP signal to uWSGI.

```bash
sudo service flask-uwsgi reload
```

References
---
- [uWSGI Quickstart](http://uwsgi-docs.readthedocs.org/en/latest/WSGIquickstart.html)
- [uWSGI Configuration](http://uwsgi-docs.readthedocs.org/en/latest/Configuration.html)
- [uWSGI Upstart](http://uwsgi-docs.readthedocs.org/en/latest/Upstart.html)
