============
Installation
============

Requirements
------------

* Docker daemon
* GitHub application key (see https://github.com/settings/developers)
* GitHub access token (https://github.com/settings/tokens)

* Edit `etc/provision.yml` to setup public key fetching of `.ssh/known_hosts`
    * Fixed known_host keys can be stored inside `etc/known_hosts` folder and can be generated with `ssh-keyscan -H HOSTNAME > etc/known_hosts/HOSTNAME` (very secure)
    * All hosts without stored known_host keys will be automatically fetched when Dockerfile build is running (less secure)
* Add ssh keys to `ssh` (will be deployed to `/home/application/.ssh`) or generate an new one with `make ssh-key`
* Run `docker-compose up -d` or `make restart`


Installation
------------

Checkout from git::

    # Checkout master (stable) branch
    git clone https://github.com/webdevops/samson-deployment.git samson-deployment
    cd samson-deployment


Setup Samson
------------


Setup ``conf/samson.conf`` file with following informations:

* ``DEFAULT_URL`` with application URL (GitHub will redirect users after login to this URL)
* ``GITHUB_CLIENT_ID`` and ``GITHUB_SECRET`` (GitHub application credentials)
* ``GITHUB_TOKEN`` (GitHub access token)
* ``SECRET_TOKEN`` with random string with 128 chars, can be generated with::

    openssl rand -hex 128| head -c 128

For building the Docker images locally you can use ``make all`` to start the build process.
There are multiple targets inside the Makefile for building only specific images eg. ``make webdevops/apache``


Setup SSH known hosts
---------------------

For deployment it's necessary to maintain valid ssh public keys inside ``~/.ssh/known_hosts`` file inside the running
Docker container. This deployment doesn't disable known hosts checking because of security issues because it allows
MitM attacks.

Inside ``etc/provision.yml`` the hostnames of ssh servers can be setup and the ssh public server keys will automatically
fetched and added to ``~/.ssh/known_hosts`` inside the Docker container.

Because automatic fetching ssh public host keys can be unsecure it's also possible to add a file with prefetched keys. As
example there are some server keys from public servers (github, gitlab, bitbucket) configured.

Setup SSH public and private key
--------------------------------

Add public and private key to ``ssh/``, this directory will be copied into ``~application/.ssh`` into the Docker container.


SSH advanced configuration
--------------------------

In ``ssh/config`` it's possible to set advanced ssh configuration options eg. different SSH ports or shorter ssh server
hostnames.

It's also possible to setup ssh proxy server to reach server behind another server with automatic ssh proxy connection::

    Host ssh-gateway
        Hostname ssh-gateway.example.com
        User foo

    Host server-behind-gateway
        Hostname server-behind-ssh-gateway.example.com
        User     root
        ProxyCommand ssh ssh-gateway -W %h:%p

