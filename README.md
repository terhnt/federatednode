[![Slack Status](http://slack.unoparty.io/badge.svg)](http://slack.unoparty.io)

**All-in-one Unoparty Federated Node Build System (`federatednode-uno`)**

Allows one to easily build a unoparty-server, `unoblock` and/or Unowallet system, with all required components. Uses Docker and Docker compose.

See official counterparty federatednode-uno documentation at <http://counterparty.io/docs/federated_node/>.

Please issue any/all pull requests against **develop**, not **master**.

-------------------------------------------------------------

# LINUX SETUP

## PRE INSTALLATION

(Instructions are provided for Ubuntu Linux. Other Linuxes will be similar. Use a sudo-er account, but not root)

**Update system & install dependencies**

Make sure you have Python 3.5. (Ubuntu 14.04 for instance uses Python 3.4 by default), but 16.04 uses 3.5. If you have an Ubuntu version older than 3.4, you can update your Python with these [instructions](http://askubuntu.com/a/682875).

```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get -y install git curl coreutils
```

Install docker-ce and docker-compose (see here for more [info](https://docs.docker.com/compose/install/), here we use v1.16.1):
```
sudo -i # become root
curl -fsSL https://get.docker.com/ | sh
curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
exit # leave root shell
```

## INSTALLATION

On Linux and OS X, install as a non-root sudo-er from home directory.

**Clone and check out the code**

On all OS, clone federatednode-uno repo and enter cloned directory:

```
git clone https://github.com/terhnt/federatednode-uno.git
cd federatednode-uno
```

On Linux and OS X:

```
sudo ln -sf `pwd`/unonode.py /usr/local/bin/unonode
```

**Build and link the containers**

Run the following command:

```
unonode install <CONFIG> <BRANCH>
```

Where ```<CONFIG>``` is one of the following:


- ```base``` if you want to run ```unoparty-server``` and ```unobtaniumd``` only
- ```unoblock``` if you want to run everything in base, with the addition of ```unoblock``` and its dependencies (```mongodb``` and ```redis```)
- ```full``` if you would like to run a __**full federated node configuration**__, which is all services on the list above

And where ```<BRANCH>``` is one of the following:

- ```master``` (stable and recommended)
- ```develop``` (cutting edge, likely with bugs - Most likely not working)

For example:

```
# install a base configuration for the master branch
unonode install base master

# install a full configuration for the develop branch
unonode install full develop
```

In some cases (slow host, limited bandwidth), you may experience a failure to install due to download timeouts which happen because of network unstability. In that case consider changing Docker’s ```max-concurrent-downloads``` value to 1 or 2 from default 3. To do that create a custom ```/etc/docker/daemon.json``` daemon options file and restart Docker service.

As mentioned earlier, the install script may stop if ports used by Federated Node services are used by other applications. While it is not recommended to run Federated Node alongside production services, small changes can make the evaluation of Federated Node easier. For example you may change ports used by existing applications (or disable said applications) or run Federated Node inside of a virtual machine.

For example, the original mongodb can be reconfigured to listen on port 28018 and unoblock’s mongodb can use the default port 27017. The Federated Node install script makes it possible to specify the interface used by its mongodb container (example below), but it currently does not have the ability to do this for other services or get around port conflicts.

```
unonode install --mongodb-interface 127.0.0.2 unoblock master
```

**Wait for initial sync**

After installation, the services will be automatically started. To check the status, issue:
```
unonode ps
```

If you have existing instances of Unobtanium Core (either mainnet or testnet), at this point you could stop all services listed in ```unonode ps``` output, change configuration files (of unoparty and unoblock, for example) and point them to your existing Unobtanium Core. Configuration files can be found in various service directories located under federatednode-uno/config.

Once the containers are installed and running, keep in mind that it will take some time for ```unobtaniumd``` to download the blockchain data. Once this is done, ```unoparty-server``` will fully start and sync, followed by ```unoblock``` (if in use). At that point, the server will be usable.

You may check the sync status by tailing the appropriate service logs, e.g. for Unobtanium Core and unoparty server on mainnet:

```
unonode tail unobtanium
unonode tail unoparty
```

**Access the system**
Once running, the system listens on the following ports:
- ```unoparty-server```: 4120/tcp (mainnet), 14120/tcp (testnet)
- ```unoblock```: 4121/tcp (mainnet), 14121/tcp (testnet)

For ```unoparty-server```, use RPC username unobtaniumrpc and default password rpc.
If ```unowallet``` is installed, access to the following URLs will be possible:

- ```http://<host>/``` — directs to ```https```
- ```https://<host>/``` - main production URL (uses minified JS/CSS)
- ```https://<host>/src/``` - development URL (uses un-minified JS/CSS)

## Post-installation tasks

Ensure that your firewall software is enabled. If you want to provide access from external systems, you can allow through some or all of the [appropriate ports](https://counterparty.io/docs/federated_node/#accessing). In addition, if you are running a node in a production scenario, it is recommended that you properly secure it.

You may also want to tighten ownership and permissions on all conf files in federatednode-uno/config subdirectories, but keep in mind that you should be the only user with access to the operating system that runs Federated Node containers: Federated Node is not designed for shared OS environments.

**Ubuntu Linux**
Ubuntu Linux users can optionally run a little script that will issue a number of commands to assist with securing their systems:
```
cd extras/host_security
sudo ./run.py
```

Note that this script will make several modifications to your host system as it runs. Please review what it does here before using it.

If you expect to run a busy Federated Node that requires unoblock, you can consider making the following performance tweaks for mongodb and redis. Please do not make these changes to the host if you’re not comfortable with them because they impact not only Docker but the entire OS.


- Disable huge memory pages (for redis and mongodb): on Ubuntu 16.04 add ```echo "never" > /sys/kernel/mm/transparent_hugepage/enabled``` to /etc/rc.local and run ```sudo systemctl enable rc-local.service```. Reboot and check with ```cat /sys/kernel/mm/transparent_hugepage/enabled``` (expected setting: [never]).
- Edit ```/etc/sysctl.conf``` (for redis): add ```net.core.somaxconn = 511``` and ```vm.overcommit_memory = 1``` and run ```sudo sysctl -p```.

## Administration

**Checking status**
To check the status of the containers, run:
```
unonode ps
```

**Modifying configurations**

Configuration files for the ```unobtanium```, ```unoparty``` and ```unoblock``` services are stored under ```federatednode-uno/config/``` and may be freely edited. The various locations are as follows:

- ```unobtanium```: See ```federatednode-uno/config/unobtanium/unobtanium.conf```
- ```unobtanium-testnet```: See ```federatednode-uno/config/unobtanium/unobtanium.testnet.conf```
- ```unoparty```: See ```federatednode-uno/config/unoparty/server.conf```
- ```unoparty-testnet```: See ```federatednode-uno/config/unoparty/server.testnet.conf```
- ```unoblock```: See ```federatednode-uno/config/unoblock/server.conf```
- ```unoblock-testnet```: See ```federatednode-uno/config/unoblock/server.testnet.conf```
- ```redis```: shared service used for both mainnet and testnet
- ```mongodb```: shared service used for both mainnet and testnet

Remember: once done editing a configuration file, you must ```restart``` the corresponding service. Also, please don’t change port or usernames/passwords if the configuration files unless you know what you are doing (as the services are coded to work together smoothly with specific values).

For example, a user with base setup (Unobtanium Core & Unoparty Server) could make Unoparty use existing Unobtanium Core by changing configuration files found under federatednode-uno/config/unoparty/ (```backend-connect``` in Unoparty server configuration files and ```wallet-connect``` in client configuration files.) At this point Unobtanium Core (mainnet and/or testnet) container(s) could be stopped and unoparty server container restarted. If your existing Unobtanium Server allows RPC connections, with proper settings and correct RPC credentials in their configuration files, unoparty (server), unoblock and unowallet can all use it so that you don’t have to run unobtanium or unobtanium-testnet container.

**Viewing/working with stored data**

The various services use [Docker named volumes](https://docs.docker.com/engine/tutorials/dockervolumes/) to store data that is meant to be persistent:

- ```unobtanium``` and ```unobtanium-testnet```: Stores blockchain data in the ```federatednode-uno_unobtanium-data``` volume
- ```addrindexrs_uno``` and ```addrindexrs_uno-testnet```: Stores index data in the ```federatednode-uno_addrindexrs_uno-data``` volume
- ```unoparty``` and ```unoparty-testnet```: Stores Unoparty databases in the ```federatednode-uno_unoparty-data``` volume
- ```unoblock``` and ```unoblock-testnet```: Stores Unoblock asset info (images), etc in the ```federatednode-uno_unoblock-data``` volume
- ```mongodb```: Stores the databases for unoblock and unoblock-testnet in the ```federatednode-uno_mongodb-data``` volume

Use docker ```volume inspect <volume-name>``` to display volume location. See docker ```volume --help``` for help on how to interact with Docker volumes.

**Viewing logs**

To tail the logs, use the following command:

```
unonode tail <service>
```

Or, to view the entire log, run:

```unonode logs <service>```

Where ```<service>``` may be one the following, or blank to tail all services:


- ```unoparty``` (unoparty-server mainnet)
- ```unoblock``` (unoblock mainnet)
- ```unobtanium``` (unobtanium mainnet)
- ```addrindexrs_uno``` (addrindexrs_uno mainnet)
- ```armory_utxsvr``` (armory_utxsvr mainnet)
- ```unoparty-testnet```
- ```unoblock-testnet```
- ```unobtanium-testnet```
- ```addrindexrs_uno-testnet```
- ```armory_utxsvr-testnet```
- ```unowallet```

**Stopping and restarting containers**

```
unonode stop <service>
unonode start <service>
unonode restart <service>
```

Where ```<service>``` is one of the service names listed [above](https://counterparty.io/docs/federated_node/#servicenames), or blank for all services.

Note that redis and mongodb are shared services and need to run if either (mainnet or testnet) unoblock container is running and shut down only if both unoblock containers are not running.

**Issuing a single shell command**

```
unonode exec <service> <CMD>
```

Where ```<service>``` is one of the service names listed above, and ```<CMD>``` is an arbitrary shell command.

For example:

```
unonode exec unoparty unoparty-client send --source=uLoY132tBNEKDD3CRkrAen8Jnckjy1gXZR --destination=uZL4y6ysx2s4wWUDxMHYJ8YeqNJA1nWAGV --quantity=1.5 --asset=XUP
unonode exec unobtanium-testnet unobtanium-cli getpeerinfo
unonode exec unpblock ls /root
```

**Getting a shell in a conainer**

```
unonode shell <service>
```

Where ```<service>``` is one of the service names listed [above](https://counterparty.io/docs/federated_node/#servicenames)

## Updating, rebuilding, uninstalling

To pull the newest software from the git repositories and restart the appropriate daemon, issue the following command:

```
unonode update <service>
```

Where ```<service>``` is one of the following, or blank for all applicable services:

- ```unoparty```
- ```unoparty-testnet```
- ```unoblock```
- ```unoblock-testnet```
- ```armory_utxsvr```
- ```armory_utxsvr-testnet```
- ```unowallet```


**Reparsing blockchain data**

Both ```unoparty-server``` and ```unoblock``` read in blockchain data and construct their own internal databases. To reset these databases and trigger a reparse of this blockchain data for one of the services, run:

```
unonode reparse <service>
```

Where service is ```unoparty```, ```unoparty-testnet```, ```unoblock```, or ```unoblock-testnet```.

**Rebuilding a service container**

As a more extensive option, if you want to remove, rebuild and reinstall a container (downloading the newest container image/```Dockerfile``` and utilizing that):

```
unonode rebuild <service>
```

Where ```<service>``` is one of the service names listed [earlier](https://counterparty.io/docs/federated_node/#servicenames), or blank for all services. Note that you are just looking to update the source code and restart the service, ```update``` is a better option.

**Uninstalling**

To uninstall the entire unonode setup, run:

```
unonode uninstall
```

## Component development

The system allows for easy development and modification of the Unoparty software components. To do so, simply update code in the directories under ```federatednode-uno/src/``` as you see fit. These directories are mapped into the appropriate containers, overlaying (overriding) the source code that the container ships with. This, along with symlinked (develop) Python package installations makes it possible to work on code in-place, with just a service restart necessary to have the changes take effect.

Once done updating the source code for a particular service, issue the following command(s) to restart the container with the new code:

```
unonode restart <service>
```

Where ```<service>``` is one of the services mentioned [here](https://counterparty.io/docs/federated_node/#servicenames_code).

**Other Developer Notes**

- To run the ```unoparty-lib``` test suite, execute:

```
unonode exec unoparty "cd /unoparty-lib/unopartylib; py.test --verbose --skiptestbook=all --cov-config=../.coveragerc --cov-report=term-missing --cov=./"
```

- If you are working on ```unowallet```, you should browse the system using the ```/src/``` subdirectory (e.g. ```https://myunowallet.bla/src/```). This avoids using precompiled sources. Once you are happy with your changes and ready to make them available to everyone that hits the server, run unonode update unowallet, which will pull the newest repo code and repackage the web assets so that the code updates are then active from ```https://myunowallet.bla/```.

- Note that when you install the federated node system, HTTPS repository URLs are used by default for all of the repositories checked out under ```src``` by ```unonode.py```. To use SSH URIs instead, specify the ```--use-ssh-uris``` to the ```unonode install``` command.

**Counterwallet-Specific**

If you are setting up a Unowallet server, you will next need to create a ```unowallet.conf.json``` configuration file. Instructions for doing that are detailed in the __**Unowallet Configuration File**__ section later in this document. Once creating this file, open up a web browser, and go to the IP address/hostname of the server. You will then be presented to accept your self-signed SSL certificate, and after doing that, should see the Unowallet login screen.

**Getting a SSL Certificate**
By default, the system is set up to use a self-signed SSL certificate. If you are hosting your services for others, you should get your own SSL certificate from your DNS registrar so that your users don’t see a certificate warning when they visit your site.

Once you have that certificate, create a nginx-compatible ```.pem``` file. Copy that ```.pem``` file to ```federatednode-uno/config/unowallet/ssl/unowallet.pem``` and the cooresponding certificate ```.key``` file to ```federatednode-uno/config/unowallet/ssl/unowallet.key```. (Note that there will be a ```unowallet.key``` and ```unowallet.pem``` file already there, which are the default, self-signed certificates, and can be safely overridden.) Then, restart the ```unowallet``` service for the new certificate to take effect.

**Monitoring the Server**

To monitor the server, you can use a 3rd-party service such as [Pingdom](http://www.pingdom.com/) or [StatusCake](http://statuscake.com/). The federated node allows these (and any other monitoring service) to query the basic status of the Federated Node via making a HTTP GET call to one of the following URLs:

- ```/_api/``` (for mainnet)
- ```/_t_api/``` (for testnet)

If all services are up, a HTTP 200 response with the following data will be returned:

```
{"unoparty-server": "OK", "unoblock_ver": "1.3.0", "unoparty-server_ver": "9.31.0", "unoblock": "OK",
"unoblock_check_elapsed": 0.0039348602294921875, "unoparty-server_last_block": {
"block_hash": "0000000000000000313c4708da5b676f453b41d566832f80809bc4cb141ab2cd", "block_index": 311234,
"block_time": 1405638212}, "local_online_users": 7, "unoarty-server_check_elapsed": 0.003687143325805664,
"unoblock_error": null, "unoparty-server_last_message_index": 91865}
```

Note the ```"unoparty-server": "OK"``` and ```"unoblock": "OK"``` items.

If all services but ```unoparty-server``` are up, a HTTP 500 response with ```"unoparty-server": "NOT OK"```, for instance.

If ```unoblock``` is not working properly, ```nginx``` will return a HTTP 503 (Gateway unavailable) or 500 response.

If ```nginx``` is not working properly, either a HTTP 5xx response, or no response at all (i.e. timeout) will be returned.

**Creating a configuration file**

Unowallet can be configured via editing the ```unowallet.conf.json``` file, via issuing the following command:
```
sudo docker exec -it federatednode-uno_unowallet_1 vim /unowallet/unowallet.conf.json
```

This file will contain a valid JSON-formatted object, containing an a number of possible configuration properties. For example::

```
{
  "servers": [ "unoblock1.mydomain.com", "unoblock2.mydomain.com", "unoblock3.mydomain.com" ],
  "forceTestnet": true,
  "googleAnalyticsUA": "UA-48454783-2",
  "googleAnalyticsUA-testnet": "UA-48454783-4",
  "rollbarAccessToken": "39d23b5a512f4169c98fc922f0d1b121Click to send altcoins to this UNO address ",
  "disabledFeatures": ["betting"],
  "restrictedAreas": {
    "pages/betting.html": ["US"],
    "pages/openbets.html": ["US"],
    "pages/matchedbets.html": ["US"],
    "dividend": ["US"]
  },
}
```

Here’s a description of the possible fields:

**Required fields:**

- **servers:** Unowallet should work out-of-the-box in a scenario where you have a single Unoblock Federated Node that both hosts the static site content, as well as the backend Unoblock API services. However, Unowallet can also be set up to work in MultiAPI mode, where it can query more than one server (to allow for both redundancy and load balancing). To do this, set this ```servers``` parameter as a list of multiple server URIs. Each URI can have a ```http://``` or ```https://``` prefix (we strongly recommend using HTTPS), and the strings must __**not**__ end in a slash (just leave it off). If the server hostname does not start with ```http://``` or ```https://```, then ```https://``` is assumed.

If you just want to use the current server (and don’t have a multi-server setup), just specify this as ```[]``` (empty list).*

**Optional fields:**


- **forceTestnet**: Set to true to always use testnet (not requiring ‘testnet’ in the FQDN, or the ‘?testnet=1’ parameter in the URL.
- **googleAnalyticsUA** / **googleAnalyticsUA-testnet**: Set to enable google analytics for mainnet/testnet. You must have a google analytics account.
- **rollbarAccessToken**: Set to enable client-side error tracking via rollbar.com. Must have a rollbar account.
- **disabledFeatures**: Set to a list of zero or more features to disable in the UI. Possible features are: ```betting```, ```dividend```, ```exchange```, ```leaderboard```, ```portfolio```, ```stats``` and ```history```. Normally this can just be ```[]``` (an empty list) to not disable anything.
    restrictedAreas: Set to an object containing a specific page path as the key (or “dividend” for dividend functionality), and a list of one or more ISO 2-letter country codes as the key value, to allow for country-level blacklisting of pages/features.

Once done, save this file and make sure it exists on all servers you are hosting ```Unowallet``` static content on, and restart the unowallet service. Now, when you go to your Unowallet site, the server will read in this file immediately after loading the page, and set the list of backend API hosts from it automatically.
