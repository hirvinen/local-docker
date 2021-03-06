# Local-docker

**NEVER EVER EVER USE THESE `docker-compose*.yml` FILES IN PRODUCTION**.

This is a template for local development, mainly targeted for Drupal.

In short: Drupal's all PHP lives in `./app` -folder, and `index.php` -
the docroot - is in `./app/web` -folder. All containers are properly
versioned, and none of the containers are updated automatically for
projects local environment's long term consistency.

Everything happens in containers, and command `./ld` is the main tool
for configuring local, booting it up and so on.

## Requirements

Your laptop should have
[Docker and Docker compose](https://docs.docker.com/compose/) as well
as [Docker-sync](https://docker-sync.readthedocs.io). Install
[Homebrew](https://brew.sh/) if you have not done it already, and run
this on you host (in any directory):

    $ brew install cask 
    $ brew cask install docker-edge # edge channel OR
    $ brew cask install docker # stable channel
    $ sudo gem install docker-sync

You may have the Docker already installed, in which case
[Homebrew](https://brew.sh/) will tell you.

## Usage

Main usage is done using a `ld.sh `script: `./ld`. Executing the script
without no arguments gives you command list.

### Start using local-docker

This tool can be used both as a base for new Drupal 8 projects and as a
local development tool for Skeleton based projects (known by Exove
developers).

#### New project
Clone this repository as your project folder, reconfigure Git repository
and start the initialisation.

    $ git clone git@github.com:Exove/local-docker.git my-project
    $ cd my-project
    # Reconfigure first git remote 'origin' (so you do not push to
    # local-docker -repository by mistake).
    $ git remote set-url origin ssh://git.example.com/my-project.git
    # Verify you have correct remote url's.
    $ git remote -v
    # Create a disconnected temporary branch w/ no commits. This new 
    # branch is not connected to the old repository branches.
    $ git checkout --orphan master-new
    # Commit clean project base to the new and still empty branch.
    $ git commit -am 'Initial commit for my-project from local-docker'
    # Delete old master branch (with the full commit history), and
    # rename your temporary branch to master.
    $ git branch -D master
    $ git branch -m master-new master
    $ git push
    $ ./ld init

#### Existing project
 
Copy all of this repository on top of your current project.

    db_dumps/
    docker/
    .env.example 
    .env.local.example 
    .gitignore.example # (or copy rules to your existing .gitignore)
    ld.sh 
    ld  # (ensure you have this symlinking to correct ld.sh)

Open your favourite terminal, type `./ld` and hit \[ENTER]. If you get
an error, ensure `ld.sh` has execute permission:

    $ chmod 0744 ld.sh

Initial setup asks if you some generic information. 

    $ ./ld init

If you are applying `local-docker` on a Skeleton -based project, see
"Skeleton" -section .

### Local domains

`./ld init` sets up your local IP addresses and domains. You'll be asked
for a local development domain among ohter things, and `local-docker`
will write a `/etc/hosts` -record for you and maintain localhost IP
address aliases.

Some development tools and other services are accessible via their own
subdomains: 

- main project - [http://example.com]() and [http://www.example.com]()
- MySQL database - [mysql://$LOCAL_IP:3306](mysql://$LOCAL_IP:3306)
  (`LOCAL_IP` address is defined in the `.env` file)
- Adminer (Web UI for MySQL) -
  [http://**adminer**.example.com](http://adminer.example.com)
- Mailhog (catches **all** emails) -
  [http://**mailhog**.example.com](http://mailhog.example.com)
- Solr (search index web UI)) -
  [http://**solr**.example.com](http://solr.example.com)
- Traefik (Reverse proxy for handling HTTP traffic) -
  [http://**traefik**.example.com](http://traefik.example.com)
- Whoami (**only** request headers ) -
  [http://**whoami**.example.com](http://whoami.example.com)

#### Install Drupal

Once the local is up and running you can install Drupal.

**Note:** MySQL is accessible in `db` instead of `localhost`.

1. Using Drush 

        $ ./ld drush si -y minimal --db-url=mysql://drupal:drupal@db/drupal

2. Using browser: 

    Drupal is not yet installed you'll be redirected to
    `www.example.com/core/install.php`.
    
    Install Drupal as usual. Be default database -container (`db`) has one
    database (`drupal`) and credentials for accessing (`drupal`/`drupal`).
    **Database hostname must be `db`** (database container name). Docker
    connects containers internally using container names.
    
    If you need more databases or need to manage anything inside
    Drupal's database, you can 
    
    1. connect to `db` container either via shell
    
            $ docker-compose exec db sh
    
    2. or use Adminer with your browser:
       [http://adminer.example.com](http://adminer.example.com)
    
   3.  or use your favourite SQL GUI app (SequelPro or similar), and
       connect using LOCAL_IP (see `.env` file, `127.0.X.Y`),
       default port `3306`, username `root` and password from your
       `.env` file, `MYSQL_ROOT_PASSWORD`.
 
#### Skeleton

If you are applying Local-docker on a Skeleton based project start by
copying all things mentioned in "Start using local-docker" -section on
top of your project repository and copy all environment variables from.
`.env.example` to your own `.env` file.

    $ ./ld init skeleton

After some configuration your codebase is built, and Docker volumes
(including volumes used by `docker-sync`) according general Skeleton
structure.

Drupal should also connect to correct database host. This is usually
done via your site's `settings.php` file. Where Skeleton may use
`localhost` as the database host. `local-docker` uses `db` container,
and by default credentials `drupal`/`drupal` with database named
`drupal` (simple!). Example for Drupal 8:

    <?php 
      $databases['default']['default'] = [
        'database' => 'drupal',
        'username' => 'drupal',
        'password' => 'drupal',
        'prefix' => '',
        'host' => 'db',   // <== IMPORTANT!
        'port' => '3306',
        'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
        'driver' => 'mysql',
      ];

However, editing `settings.php` manually for database connection is
usually necessary only if you are applying `local-docker` on top of an
existing Skeleton -project.

### Daily usage

Execute `./ld` to see some options to control your local docker stack.
Behind the scenes script starts and stops files syncing and containers.

More importantly using it helps you to not delete database by mistake,
but executes (and restores) database backups in `db_dumps/` -folder,
keeping most recent as the restorable dump. 

#### Initial start

It is recommended to start the project first time with:

    $ ./ld init

and answer some question to set up things properly.

#### Start local

Pulls, builds and starts all containers.

    $ ./ld up

#### Stop local

Put local to sleep (no DB dump created, but databases won't be
immediately destroyed either).

    $ ./ld stop 

Stops local, create DB dump.

    $ ./ld down

Note that files sync must be started in order to start other containers,
and it keeps 1-n pcs of containers running when it is started. `./ld`
commands take care of that, too.

#### Watch filesync logs

It may be helpful to keep eye on files sync logs.
 
    $ docker-sync logs -f

#### Create a snapshot/backup of database

This is done automatically when you stop or destroy your containers.

    $ ./ld dump

#### Restore (old) db backup
 
 Put a file in `db_dumps/` -folder and create a symlink pointing to it:
 
    $ ln -s MY_GZIPPED_MYSQLDUMP_FILE.sql.gz ./db_dumps/db-container--FULL---LATEST.sql.gz
    $ ./ld restore [PATH-TO-THE-GZIP-DUMP]


#### Composer, Drush, Drupal

Composer, Drush or Drupal console are aliased inside `php` container as
well as commands in `./ld [drush|drupal|composer]`.

    $ docker-compose exec php bash
    /var/www # drush status
    /var/www # composer require drupal/pathauto:^1
    /var/www # drupal ce --exclude-config-hash --directory=../config/sync

These commands are funcionality-wise the same:

    $ ./ld drush status
    $ ./ld composer require drupal/pathauto:^1
    $ ./ld drupal ce --exclude-config-hash --directory=../config/sync

You can also execute commands directly in the shell:

    $ docker-compose exec php bash -c "drush status"
    # OR
    $ ./ld drush status

#### Compile CSS

Look at the `nodejs` -container in `docker-compose*.yml`, correct file
paths and enable it. Launch the project again to get the container
booted up.

You can expose nodejs container logs with:

    $ docker-compose logs -f nodejs

#### Xdebug

`php` -container has Xdebug up and running, but not turned on by
default. You can read Xdebug value, and toggle it on/off with this
command:

    $ ./ld xdebug [1|0]

When you load your PHP application, or run Drush / Drupal console
commands `php` container tries to connect to your IDE:

    xdebug.remote_port = 9010
    ; Docker for Desktop (on OSX at least) maps host.docker.internal to
    ; host machine. 
    xdebug.remote_host = host.docker.internal

This Xdebug configuration is initially set in the base image this
project is using (`xoxoxo/php-container`). However this can be
overridden for example in
[95-drupal-development.ini file (PHP 7.2)](./docker/build/php7.2//conf.d/95-drupal-development.ini)
, and Xdebug's full config can be checked either from Drupal
(`admin/reports/status/php`) or with command

    $ docker-compose exec php sh -c 'exec /usr/local/bin/php -i |grep xdebug'

**Note:** You should set your IDE to use port `9010` for Xdebug in order
for IDE to setup a Xdebug listener to the port Xdebug is trying to
connect to on your host. Port `9010` is being used to avoid collision
with possible running `php-fpm` on the host machine port `9000`.

#### Convenience "alias" for ./ld.sh

If you find the `./` prefix tedious when using ld, you could add the following
alias to your shell startup files. In order to not interfere with `/usr/bin/ld`,
`ld` as alias should be avoided.

    function lld() { if [[ -x "$(pwd)/ld.sh" ]] ; then "$(pwd)/ld.sh" $@; else echo "ld.sh not found in current directory"; fi }

Then, you can use `lld` instead of `./ld` or `./ld.sh`

#### Update local-docker itself

Since project repository is detached from the `local-docker` repository
there is a script to get the new updates for the local-docker itself.

    $ docker/scripts/self-update.sh [RELEASE]

Script defaults to using the latest release. All available releases can
be seen in
[https://github.com/Exove/local-docker/releases](https://github.com/Exove/local-docker/releases).

## Projects in parallel

`local-docker` isolates local projects behind IP aliases (alias to the
loopback interface) and therefore you can run several projects in
parallel.

In case there are port collisions first thing to check is you have
`.env` file with `LOCAL_IP` set to something other than `127.0.0.1`.
`./ld init` sets up a random IP address from range `127.0.0.0/16`.

You can change IP address by putting your local down (`./ld down`),
changing the IP address value, and starting your local again.

You can check currently set aliases using command:  
` ifconfig lo0 | grep netmask | grep -v '127.0.0.1'`

### Local ports

Docker exposes services to host using ports `:80` and `:3306`. Exposed
ports are bind to `LOCAL_IP` to restrict outside access (from your
current network) and to make parallel running of your projects possible.

## Customize ld -script variables

Local-docker configuration is defined in `./ld.config`  file, which is
created during `./ld init` process. The file should be committed to
project repository and be shared among all developers. 

Local overrides can be set via `.env.local`  file, for example to have
different local development domain or IP address. This file should not
be committed to project repository but be considered private.

Variables present in the `.env.local` file will override project level
configurations from `.env`.

These file should contain key=value -pairs, such as

    # Comments start with a hash.
    MYSQL_ROOT_PASSWORD=some_password
    ANOTHER_KEY=the-value


## What's in the package?

Behind scenes `local-docker` uses Docker. Recipe of what is launched is
in `docker-compose.yml` (or for the DEVELOPMENT server
`docker-compose.dev-vm.yml`).

To overcome the known technical limitations (ie. nerve wrecking local
drupal site slowness) `docker-sync` is being used. It sets up
[intermediate containers and hides the OSX filesystem incompatibility](https://docker-sync.readthedocs.io/en/latest/advanced/how-it-works.html)
there. 

### Pros and cons

**Con** is that the initial sync after launching the local takes a bit
of time (up to a few minutes). Once that is done local is ... ready.
Files syncing between host and docker containers may take a few seconds,
but running `docker-sync logs -f` on a terminal will expose what and
when is being synced. It is also the fastest tested flavor of various
ways to share folders between host and containers (including NFS
mounts).

**Pros** No rotten local development environments ever again! At least
for the handful of years a head of you. The trick is local environment
is not **built** locally, but loaded from the Docker hub as-is (apart
for some minor *config* adjustments).

## ISSUES 

#### Local-docker does not start


1. `Bind for 0.0.0.0:80: unexpected error (Failure EADDRINUSE)`
  
   There is some other application that reserves port :80 on your
   localhost. Turn it off.

   Turn of yor local (on Mac) web server (Apache, Nginx, whatever):

       $ sudo apachectl stop && sudo service nginx stop

    Turn off your local MySQL (on Mac):  
    > System preferences -> MySQL ->stop
    
    Check also "Projects in parallel" -section. Other projects or tools
    may have acclaimed ports your project is trying to use.

#### Local files not getting synced

If you feel file sync is not working properly, start by checking logs
and editing some files that should be synced:

    $ docker-sync logs -f

Docker-sync may get stuck on host or inside the container. Sometimes
stuck can be 'released' nudging the sync container a bit:

    $ docker-sync start 

If this does not help (chekc `docker-sync logs -f`) try cleaning the
syncs:

    $ docker-sync stop && docker-sync clean && docker-sync start

If even this does not solve the issue, *Docker* itself may be hanging. 
none of your edits in host or in container are being synced, clean up
and restart:

1. stop file sync and local (`./ld down`) if needed
2. clean up your volumes `./ld nuke-volumes` if needed
3. start your local again `./ld up` and optionally restore database
   `./ld restore`

Optionally restart Docker, or reboot your operating system. 

The last resort is to clean up everything Docker -related. **BE WARNED**
This will delete ALL volumes in ALL Docker projects across your laptop.
 
    $ docker kill $(docker ps -q) # Stop all containers
    $ docker container prune # Remove all stopped containers
    $ docker volume prune # Remove all unused local volumes
        
If even that does not help, clean up EVERYTHING Docker -related
(downloaded images, created volumes and containers).

**BE WARNED** This will delete ALL volumes, downloaded images and
containers in ALL Docker projects across your whole laptop. Sorts of
resets everything else but Docker configuration.

    $ docker system prune --volumes

#### Docker-stack does not start

`mkmf.rb can't find header files for ruby at /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/include/ruby.h`

You have probably updated Xcode or Command Line Tools but have not yet
approved a new license.

Makes sure you have Command Line Tools installed:

    $ xcode-select --install

After initial Command Line Tools installation and updates may need to
accept the license.

Another option is to install Xdoce from the App store, launch the application
and approve the license during the initial start. **However**, for the Docker Sync full Xcode is unnecessary, Command Line Tools can handle what's needed.

If you get this error: `xcode-select: error: tool 'xcodebuild' requires
Xcode, but active developer directory
'/Library/Developer/CommandLineTools' is a command line tools instance`
you should also reset the command line tools path with
 
    $ sudo xcode-select -r

### NFS share does not work 

When you start local-docker but an error like

    ERROR: for YOUR-PROJECT_nodejs  Cannot create container for service nodejs: failed to mount local volume: mount :/Users/USERNAME/Projects/clients/YOUR-PROJECT/app:/var/lib/docker/volumes/YOUR-PROJECT_SYNCNAME/_data, data: addr=192.168.65.2,nolock,nfsvers=3: permission denied

Host OS (macOS) is preventing NFS to share parts of your filesystem.
MacOS updates tend to change the settings, and in Catalina the shareable
parts of the filesystem are behind strict access control.

Your MacOS should be set to allow sharing the folder(s) you are trying
to share but only to localhost (mind the path). Check that user id (501)
and group id (20) match your own, and also use the correct, full path to
your projects folder (`/Users/perttuehn/Projects`):

     $ id -u
     501 
     $ id -g
     20
     $ sudo -i
     Password: [YOUR-PASSWORD]
     root# echo '/Users/perttuehn/Projects -alldirs -mapall=501:20 localhost' >> /etc/exports
     root# nfsd restart; exit

NOTE: If your project is located under `~/Documents` you must also grant
"Full Disk Access" to `/sbin/nfsd` in Settings > Security & Privacy >
Privacy.

![Granting a full disk access to /sbin/nfsd](./docker/media/macos-security-privacy-nfsd-access.png)
"Granting a full disk access to /sbin/nfsd"

## Issues with installing gems / Unable to download data from https://rubygems.org/ [For Mac users]

     $ sudo gem install docker-sync # docker-sync is just an example
     ERROR:  Could not find a valid gem 'docker-sync' (>= 0), here is why:
     Unable to download data from https://rubygems.org/ - SSL_connect returned=1 errno=0 state=SSLv2/v3 read server hello A: tlsv1 alert protocol version (https://api.rubygems.org/specs.4.8.gz)

If you are encountering the error above when trying to install a gem, simple workaround is to explicitly set the installation path and source (with http protocol):

     $ sudo gem install -n /usr/local/bin docker-sync --source http://rubygems.org

## Why my favourite feature is not there?

This is the initial version of the local-docker. Redis, Solr and others
are coming once the need arises.

**Asking for help is highly recommended, and pull requests even more
so.**