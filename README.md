# OJS (Open Journal Systems) - PKP - Container/Docker


| **IMPORTANT:** This repository is still beta, so it can't be used in production environments. |
|:---------------------------------------------------------:|
| We are actively working to release a stable version soon. Keep tuned. |


Open Journal Systems (OJS) is a journal management and publishing system that has been developed by the [Public Knowledge Project](https://pkp.sfu.ca/) through its federally funded efforts to expand and improve access to research.

The images in this repository are built on top of [Alpine Linux](https://alpinelinux.org/) and come in several variants, see [Versions](#versions).
This repository is a fork of the work formerly done by [Lucas Dietrich](https://github.com/lucasdiedrich/ojs).


## Demo

The fastest way to test docker-ojs is running the last stable OJS release with "Play with docker".

Just click the following link and create a user:

[![Try in PWD](https://raw.githubusercontent.com/play-with-docker/stacks/master/assets/images/button.png)](https://labs.play-with-docker.com/?stack=https://raw.githubusercontent.com/pkp/docker-ojs/master/versions/3_2_0-2/alpine/apache/php73/docker-compose.yml)

Then wait till the containers are built and click in the 8080 port link to get a fresh clean demo with the last stable ojs version.

| **TIP:** Change the version in the url if you want to test an old version. |
|:-----------------------------------------------------------:|

## How to use

If you want to run it locally or in your own server, first you need to install docker (docker-compose is also recommeneded).

For all available versions, we provide a [docker-compose](https://docs.docker.com/compose/) configuration file so you can start an OJS stack (web app and database containers) with a single command.

1. Clone this repository in your machine (if you don't like git, [download](https://github.com/pkp/docker-ojs/archive/master.zip) and unzip it):

    ```bash
	$ https://github.com/pkp/docker-ojs.git
    ```

1. Go to the directory of your OJS version of your choice:
    ```bash
    $ cd versions/3_2_0-1/alpine/apache/php73
    ```
| **TIP:** In production sites you would like to change the default configuration. The recommended way is uncommenting the environment variable sections in your docker-compose.yml and set the [environment variables](#environment-variables). |
|:---------------------------------------------------------:|

1. Run the stack:
    ```bash
    $ docker-compose up
    ```

	Docker-compose will pull images from docker Hub and do all the hard work to rise a functional OJS stack.

1. Access **http://127.0.0.1:8080** and continue through web installation process.

   Note that the database connection needs the following options:

  - **Database driver**: `mysqli`
  - **Host**: `db` (which is the name of the container in the internal Docker network)
  - **Username**: `ojs`
  - **Password**: `ojsPwd`
  - **Database name**: `ojs`
  - _Uncheck_ "Create new database"
  - _Uncheck_ "Beacon"

| **TIP:** To go through the OJS installation process automatically, set the environment variable `OJS_CLI_INSTALL=1`, and use the other environment variables to automatize the process. |
|:---------------------:|

## Building local images

Each version also includes has a file called `docker-compose-local.yml`.

This stack won't ask dockerHub for the required images, it expects you build a docker image locally. It is useful if you don't want external dependencies or you like to modify the offical Dockerfiles to fit your sepecific needs.

To do this...

1. Go to your prefered version folder and and build the image as follows:
```bash
	docker build -t local/ojs:3_2_0-2 .
```

  If something goes wrong, probably it's because you ran the former command with the wrong version number or in a folder without the local Dockerfile.

2. Once the image is built, you can run the stack locally telling compose to use the local version with the `-f`/`--file` option as follows:
```bash
docker-compose --file docker-compose-local.yml up
```


## Versions

Different OJS versions are combined with different versions of PHP (5 and 7), and different web servers ([Apache HTTP Server](https://httpd.apache.org/), [nginx](https://nginx.org/)).
_Currently, not all these combinations work! We are mostly focused in Apache. PR are welcome_

All version tags can be found at [Docker Hub Tags tab](https://hub.docker.com/r/pkpofficial/ojs/tags/).
If no webserver is mentioned in the tag, then Apache is used.

## Environment Variables

| NAME            | Default   | Info                                                   |
|:---------------:|:---------:|:------------------------------------------------------:|
| SERVERNAME      | localhost | Used to generate httpd.conf and certificate            |
| OJS_CLI_INSTALL | 0         | Used to install ojs automatically when start container |
| OJS_DB_HOST     | localhost | Database host                                          |
| OJS_DB_USER     | ojs       | Database username                                      |
| OJS_DB_PASSWORD | ojs       | Database password                                      |
| OJS_DB_NAME     | ojs       | Database name                                          |

_**Note:** OJS_CLI_INSTALL and certificate features are underconstruction._

## Special Volumes

Docker content is efimerous by design, but in some situations you would like
to keep some stuff persistent between docker falls (ie: database content,
upload files, plugin development...)

The version files include some of this folders, but are empty and disabled
by default. To enable them, **you only need to uncomment the volume lines in
your docker-compose.yml**.

When you run the docker-compose it will mount the volumes with persistent
data and will let you share files from your host with the container.

| Host									| Container	| Volume								| Description					|
|:-------------------------------------:|:---------:|:-------------------------------------:|:-----------------------------:|
| ./volumes/public						| ojs		| /var/www/html/public					| All public files				|
| ./volumes/private						| ojs		| /srv/files							| All private files (uploads)	|
| ./volumes/config/db.charset.conf 		| db		| /etc/mysql/conf.d/charset.cnf			| mariaDB config file 			|
| ./volumes/config/ojs.config.inc.php	| ojs		| /var/www/html/config.inc.php			| OJS config file				|
| ./volumes/config/php.custom.ini		| ojs		| /usr/local/etc/php/conf.d/custom.ini	| PHP custom.init				|
| ./volumes/config/apache.htaccess		| ojs		| /var/www/html/.htaccess				| Apache2 htaccess				|
| ./volumes/logs/app					| ojs		| /var/log/apache2						| Apache2 Logs					|
| ./volumes/logs/db 					| db 		| /var/log/mysql 						| mariaDB Logs 					|
| ./volumes/db							| db		| /var/lib/mysql						| mariaDB database content 		|
| ./volumes/migration 					| db		| /docker-entrypoint-initdb.d			| DB init folder (with SQLs)	|
| /etc/localtime						| ojs 		| /etc/localtime						| Sync clock with the host one.	|
| TBD									| ojs		| /etc/ssl/apache2/server.pem			| SSL **crt** certificate		|
| TBD									| ojs		| /etc/ssl/apache2/server.key			| SSL **key** certificate		|

You can add your own volumes. For instance, make sense for a plugin developer
or a themer to create a volume with his/her work, to keep a persistent copy in
the host of the new plugin or theme.

An alternative way of working for developers is working with his/her own local
Dockerfile that will be build to pull the plugin for his/her own repository...
but this will be significantly slower than the volume method.

## Upgrading OJS

The update process is easy and straightforward.

1. **Stop the stack** with the old OJS version (for instance "pkpofficial/ojs:2_4_8-5").
   ```bash
   $ docker-compose stop
   ```
1. **Set the new version** in docker-compose.yml.
 	Replace the old version: ```image: pkpofficial/ojs:2_4_5-2```
	with the new one:        ```image: pkpofficial/ojs:3_2_0-2```
1. **Start the container** with the new OJS version. It will pull a new image of your OJS scripts.
   ```bash
   $ docker-compose up
   ```
1. **Run the upgrade script** to upgrade the OJS database and files. Easiest was is connecting to your OJS container with [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec/) and run `ojs-upgrade` with this single line:
   ```bash
   $ docker exec -it ojs_app_journalname /usr/local/bin/ojs-upgrade
   ```
   | **TIP:** Note the name of the OJS container using `docker ps -a`. It should be something like `ojs_app_journalname`|
   |:----------------------------------------------------------------------------------------:|

Before the upgrade you will like to [diff](https://linux.die.net/man/1/diff) your `config.inc.php` with the version of the new OJS version to learn about new configuration variables.

## Apache2

Right now the only avaliable stack is Apache2.
Configuration files and volumes are thought you will work over an apache.

If you want to know how about the fastest method to set your own config,
jump to the next section "Easy way to change config stuff".

For those who like to know about the internals, we are now going to dig
a little bit more in this:

During the image building we ask the Dockerfile to copy the content of
./templates/webServers/apache/phpVersion/root folder in the root
of your container.

It means, if you like to add your specific php settings you can do it
creating a PHP configuration in `./templates/webServers/php{5,7,73}/conf.d/0-ojs.ini`
and build you own image locally.

There are some optimized variables already, you can check them within each
version directory, e.g. `/root/etc/apache2/conf.d/ojs.conf` with your virtualhost
or `/etc/supervisor.conf`... or, now you know what Dockerfile will do, you
can add your own.

Again: Note that template's /root folder is copied into the Docker image
**at build time** so, if you change it, you must rebuild the image for changes
to take effect.

### Easy way to change config stuff

If you don't want to keep you own local images, you can use the ones we
build in dockerHub and map your config files in your docker-compose.yml.

So if you like to change something in (for instance) your php settings,
you only need to created create a `./volumes/config/php.custom.ini`
outside your container and uncomment the volume in your docker-compose.yml.

Check the volumes section for a list of folders and files we think could
be useful to overwrite or extend to fit your needs.

### Restful URLs (aka. clean URLs)

By default the restful_url are enabled and Apache is already configured,
so there is no need to use index.php over url.

### SSL

By default at the start of Apache one script will check if the SSL certificate
is valid and its CN matches your SERVERNAME, if don't it will generate a new one.
The certificate can be overwritten using a volume mount (see `docker-compose.yml` file).

_**Note:** This feature is under reveiw and could change in future._


# How could I help?

## Update the compose configurations and Dockerfiles

If you want to join the docker-ojs (aka. docker4ojs) team you will like to
contribute with new Dockerfiles or docker-composes. To do this, you will need
to clone this project and send us a PR with your contributions.

Before we follow, let us clarify something:
Versions of this project fit with OJS tag versions, BUT...
if you take a look to PKP's repositories, you will notice the tag naming
syntax changed at version 3.1.2-0. We moved from "ojs-3_1_2-0" to "3_1_2-0".

So, to avoid keeping in mind at what momment we changed the syntax, in this
project we only use the modern syntax, so you cant/must refer every version
with the same notation.

Said this, let's talk about how we generate all this versions:

The files for the different OJS versions and software stacks are generated based
on a number of template files, see directory `templates`.

You can re-generate all `Dockerfile`, `docker-compose(-local).yml`, configuration
files, etc. calling the `build.sh` script.

```bash
# generate a specific version
$ ./build.sh 3_1_2-4

# generate _all_ versions
$ ./build.sh
```

## Add a new stack to the list

Let's say we like us to distribute a "nginx" stack. We have our hands full but
you know docker and nginx so... ¿how could you contribute?

1. Be sure the version number is in the versions.list file (ie: 3_2_1-0).
1. Create the required files and folders in the templates folder.
	 Basically you will need:
		- A Dockerfile template for each php version you publish (ie: dockerfile-alpine-nginx-php73.template)
		- A generic docker-compose.yml template (templates/dockerComposes/docker-compose-nginx.template)
		- A folder with all the specific configurations (templates/webServers/nginx/php73/root)
		- Extend exclude.list with the stuff you want to be removed.
	This is the hard work. Take apache as a reference and contact us if you need indications.
1. Edit build.sh to add your version to the proper variables.
	For the nginx exemple (over alpine) it should be enough extending the webServers array:
	```
	webServers=(  'apache' 'nginx' )
	```
	Modify the script if you need it to be smarter.
1. Run build script to generate all versions again:
	```bash
	$ ./build.sh
	```
1. Test your work running docker-compose with local dockerfiles.
1. Commit a PR with your new build.sh and templates (ignore the versions folder).


## License

GPL3 © [PKP](https://github.com/pkp)
