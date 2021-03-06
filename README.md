# django-calaccess-docker

A standalone Docker stack serving the California Civic Data Coalition's
applications for analyzing the California Secretary of State’s CAL-ACCESS database.

### Layers

* [Phusion's Ubuntu base image](http://phusion.github.io/baseimage-docker/)
* Apache and mod_wsgi
* MySQL
* A Django project based on [django-calaccess-project-template](https://github.com/california-civic-data-coalition/django-calaccess-project-template)
* [django-calaccess-raw-data](http://django-calaccess-raw-data.californiacivicdata.org/en/latest/)
* [django-calaccess-campaign-browser](http://django-calaccess-campaign-browser.californiacivicdata.org/en/latest/)
* A daily cron that loads the latest data from the California Secretary of State

### Settings

The following environmental options must be configured at runtime when initializing a new container.

ENV variable    | Definition
:-------------- | :---------
MYSQL_DATABASE  | The name of the MySQL database
MYSQL_USER      | The MySQL database user's name
MYSQL_PASSWORD  | The password for the MySQL user
DJANGO_USER     | The name of the Django admin super user
DJANGO_EMAIL    | The email of the Django admin super user
DJANGO_PASSWORD | The password of the Django admin super user


### Getting started

To create a container using this image, start by retriving the image from [Docker Hub](https://registry.hub.docker.com/u/ccdc/django-calaccess/).

```bash
$ sudo docker pull ccdc/django-calaccess:0.6
```

Then fire up the container with all the required environmental settings included
as options.

```bash
$ sudo docker run \
	-p 127.0.0.1:80:80 \
	--name my-calaccess \
	-e MYSQL_DATABASE=calaccess \
	-e MYSQL_USER=ccdc \
	-e MYSQL_PASSWORD=mycrazypassword \
	-e DJANGO_USER=admin \
	-e DJANGO_EMAIL=foo@bar.com \
	-e DJANGO_PASSWORD=mydjangopassword \
	-d ccdc/django-calaccess:0.6
```

Once that finishes, visit [localhost](http://localhost) in your browser to
see the stack in action.

### Deploying to Amazon Web Services via Elastic Beanstalk

First you must have an AWS account configured for Elastic Beanstalk and 
install the ``eb`` command line utility. You'll need to use ``eb init``
and ``eb create`` to get a repository ready to roll. 

Then create a ``Dockerrun.aws.json`` file that looks something like this.

```json
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "ccdc/django-calaccess:0.6",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "80"
    }
  ],
  "Logging": "/var/log/"
}
```

Create a new directory for Elastic Beanstalk configuration.

```bash
$ mkdir .ebextensions
```

Then add a file called ``01env.config`` and configure the environment variables
with your own credentials.

```yaml
option_settings:
  - option_name: MYSQL_DATABASE
    value: calaccess
  - option_name: MYSQL_USER
    value: ccdc
  - option_name: MYSQL_PASSWORD
    value: yourcrazypassword
  - option_name: DJANGO_USER
    value: admin
  - option_name: DJANGO_EMAIL
    value: foo@bar.com
  - option_name: DJANGO_PASSWORD
    value: yourdjangopassword
```

Fire away!

```bash
$ eb deploy
```

### Building the Docker image from source

Create a virtual enviroment to work inside.

```bash
$ virtualenv django-calaccess-docker
```

Jump in and turn it on.

```bash
$ cd django-calaccess-docker
$ . bin/activate
```

Clone this repository and jump in that.

```bash
$ git clone https://github.com/california-civic-data-coalition/django-calaccess-docker.git repo
$ cd repo
```

Compile the image from the Dockerfile.

```bash
$ sudo docker build -t ccdc/django-calaccess ./src/
```

Then fire up a container.

```bash
$ sudo docker run \
	-p 127.0.0.1:80:80 \
	--name dev-calaccess \
	-e MYSQL_DATABASE=calaccess \
	-e MYSQL_USER=ccdc \
	-e MYSQL_PASSWORD=mycrazypassword \
	-e DJANGO_USER=admin \
	-e DJANGO_EMAIL=foo@bar.com \
	-e DJANGO_PASSWORD=mydjangopassword \
	-d ccdc/django-calaccess
```
