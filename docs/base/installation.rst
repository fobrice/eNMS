============
Installation
============

Requirements: python 3.6+
(Earlier versions of python not supported.)

Run eNMS in test mode
---------------------

::

 # download the code from github:
 git clone https://github.com/afourmy/eNMS.git
 cd eNMS

 # install the requirements:
 pip install -r requirements.txt

Start eNMS in debugging mode :

::

 # set the FLASK_APP environment variable
 (Windows) set FLASK_APP=app.py
 (Unix) export FLASK_APP=app.py

 # set the FLASK_DEBUG environment variable
 (Windows) set FLASK_DEBUG=1
 (Unix) export FLASK_DEBUG=1

 # run the application
 flask run --host=0.0.0.0

Start eNMS with gunicorn :

::

 # start gunicorn in command-line
 gunicorn --config gunicorn.py app:app

 # or simply
 ./boot.sh


Start eNMS as a docker container :

::

 # download & run the container
 docker run -d -p 5000:5000 --name enms --restart always afourmy/enms

Once eNMS is running, you can go to http://127.0.0.1:5000, and log in with the admin account (``admin`` / ``admin``).

Run eNMS in Production (Unix only)
----------------------------------

To start eNMS in production mode, you must change the value of the environment variable "ENMS_CONFIG_MODE" to "Production".

::

 # set the ENMS_CONFIG_MODE environment variable
 export ENMS_CONFIG_MODE=Production

The Flask secret key is used for securely signing the session cookie and other security related needs.
In production mode, the secret key is not automatically set to a default value in case it is missing. Therefore, you must configure it yourself:

::

 # set the ENMS_SECRET_KEY environment variable
 export ENMS_SECRET_KEY=value-of-your-secret-key


All credentials should be stored in a Hashicorp Vault: the environement variable ``USE_VAULT`` tells eNMS that a Vault has been setup and can be used. This variable is set to ``0`` by default in debug mode, and ``1`` in production mode.
Follow the manufacturer's instructions and options for how to setup a Hashicorp Vault.

If you want to use the Vault in debug mode, you can set it to 1:
 
::

 # set the USE_VAULT environment variable
 export USE_VAULT=1

Once this is done, you must tell eNMS how to connect to the vault:

::

 # set the VAULT_ADDR environment variable
 export VAULT_ADDR=vault-address

 # set the VAULT_TOKEN environment variable
 export VAULT_TOKEN=vault-token

eNMS can also unseal the Vault automatically at start time.
This mechanism is disabled by default. To activate it, you need to:
- set the ``UNSEAL_VAULT`` environement variable to ``1``
- set the UNSEAL_VAULT_KEYx (``x`` in [1, 5]) environment variables :

::

 export UNSEAL_VAULT=1
 # set the UNSEAL_VAULT_KEYx environment variable
 export UNSEAL_VAULT_KEY1=key1
 export UNSEAL_VAULT_KEY2=key2
 etc

You also have to tell eNMS the address of your database by setting the "ENMS_DATABASE_URL" environment variable.

::

 # set the ENMS_DATABASE_URL environment variable
 export ENMS_DATABASE_URL=database-address

In case this environment variable is not set, eNMS will default to using a SQLite database.

Run eNMS with a PostgreSQL database
-----------------------------------

In production, it is advised to use a PostgreSQL database to store data. This can be hosted locally or on a remote server. 

.. note:: The installation instructions provided here have been tested to work on Ubuntu 16.04 and CentOS 7.4. The particular commands needed to install dependencies on other distributions may vary significantly.

Installation on Ubuntu
**********************

If a recent enough version of PostgreSQL is not available through your distribution's package manager, you'll need to install it from an official PostgreSQL repository.

::

 sudo apt-get update
 sudo apt-get install -y postgresql libpq-dev

Installation on Centos
**********************

Centos: CentOS 7.4 does not ship with a recent enough version of PostgreSQL, so it will need to be installed from an external repository. The instructions below show the installation of PostgreSQL 9.6.

::

 yum install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
 yum install postgresql96 postgresql96-server postgresql96-devel
 /usr/pgsql-9.6/bin/postgresql96-setup initdb

CentOS users should modify the PostgreSQL configuration to accept password-based authentication by replacing ``ident`` with ``md5`` for all host entries within ``/var/lib/pgsql/9.6/data/pg_hba.conf``. For example:

::

 host    all             all             127.0.0.1/32            md5
 host    all             all             ::1/128                 md5

Then, start the service and enable it to run at boot:

::

 systemctl start postgresql-9.6
 systemctl enable postgresql-9.6

Database creation
*****************

At a minimum, we need to create a database for eNMS and assign it a username and password for authentication. This is done with the following commands.

::

 sudo -u postgres psql -c "CREATE DATABASE enms;"
 sudo -u postgres psql -c "CREATE USER enms WITH PASSWORD 'strong-password-here';"
 sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE enms TO enms;"

You can verify that authentication works issuing the following command and providing the configured password. (Replace ``localhost`` with your database server if using a remote database.)

::

 psql -U enms -W -h localhost enms

If successful, you will enter a enms prompt. Type \q to exit.

Export PostgreSQL variables
***************************

The configuration file contains the SQL Alchemy configuration:

::

 # Database
 SQLALCHEMY_DATABASE_URI = environ.get(
     'ENMS_DATABASE_URL',
     'postgresql://{}:{}@{}:{}/{}'.format(
         environ.get('POSTGRES_USER', 'enms'),
         environ.get('POSTGRES_PASSWORD'),
         environ.get('POSTGRES_HOST', 'localhost'),
         environ.get('POSTGRES_PORT', 5432),
         environ.get('POSTGRES_DB', 'enms')
     )
 )

You need to export each variable with its value:

::

 export POSTGRES_USER=your-username
 export POSTGRES_PASSWORD=your-password
 etc...

LDAP/Active Directory Integration
*********************************

The following environment variables (with example values) control how eNMS integrates with LDAP/Active Directory for user authentication. eNMS first checks to see if the user exists locally inside eNMS. If not and if LDAP/Active Directory is enabled, eNMS tries to authenticate against LDAP/AD using the pure python ldap3 library, and if successful, that user gets added to eNMS locally.

::

  Set to 1 to enable LDAP authentication; otherwise 0:
    export USE_LDAP=1
  The LDAP Server URL (also called LDAP Provider URL):
    export LDAP_SERVER=ldap://domain.ad.company.com
  The LDAP distinguished name (DN) for the user. This gets combined inside eNMS as "domain.ad.company.com\\username" before being sent to the server.
    export LDAP_USERDN=domain.ad.company.com
  The base distinguished name (DN) subtree that is used when searching for user entries on the LDAP server. Use LDAP Data Interchange Format (LDIF) syntax for the entries.
    export LDAP_BASEDN=DC=domain,DC=ad,DC=company,DC=com
  The string to match against 'memberOf' attributes of the matched user to determine if the user is granted Admin Privileges inside eNMS.
    export LDAP_ADMIN_GROUP=company.AdminUsers

.. note:: Failure to match memberOf attribute output against LDAP_ADMIN_GROUP results in eNMS user account creation with minimum privileges. An admin user can afterwards alter that user's privileges from :guilabel:'Admin/User Management'

GIT Integration
***************

To enable sending device configs captured by configuration management, as well as service and workflow job logs, to GIT for revision control you will need to configure the following:

First, create two separate git projects in your repository. Assign a single GIT userid to have write access to both.

Additionally, the following commands need to be run to properly configure GIT in the eNMS environment. These commands populate ~/.gitconfig:

::

  git config --global user.name "git_username"
  git config --global user.email "git_username_email@company.com"
  git config --global push.default simple

Similarly, if your environment already has an SSH key created for other purposes, you will need to create a new SSH key to register with the GIT server:

::

  ssh-keygen -t rsa -f ~/.ssh/id_rsa.git

And to instruct SSH to use the new key when connecting with the GIT server, create an entry in ~/.ssh/config:

::

  Host git-server
    Hostname git-server.company.com
    IdentityFile ~/.ssh/id_rsa.git
    IdentitiesOnly yes

Additionally, the URL of the GIT server needs to be populated:
  - from the Admin Panel of the UI to store the results of services and workflows in git.
  - from the Configuration Management page to store device configurations in git.

Default Examples
----------------

By default, eNMS will create a few examples of each type of object (devices, links, services, workflows...).
If you run eNMS in production, you might want to deactivate this.

To deactivate, set the ``CREATE_EXAMPLES`` environment variable to ``0``.

::

 export CREATE_EXAMPLES=0

Logging
-------

You can configure eNMS as well as Gunicorn log level with the following environment variables

::

  export ENMS_LOG_LEVEL='CRITICAL'
  export GUNICORN_LOG_LEVEL='critical'
  export GUNICORN_ACCESS_LOG='None'
