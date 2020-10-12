# Installing the Digital Hub components with Docker Compose

This document will guide you through the configuration and installation of the components of the *Digital Hub Platform* with Docker Compose.

**Docker** and **Docker Compose** are required.

The components are the following:
- *AAC - Authentication and Authorization Control*
- *WSO2 API Manager*
- *WSO2 Data Services Server*
- *Apache NiFi*
- *Cyclotron*

In addition, we will install *Nginx*, used for its reverse-proxy capabilities.
Each tool needs a configuration file to be prepared, then it can be installed with a Compose command.

Clone the following repository and `cd` into its `docker-compose` subfolder. It contains the Docker Compose scripts and the templates for the configuration files.\
https://github.com/scc-digitalhub/platform

**AAC** is mandatory and should be installed and started **first**. **Nginx** is also necessary, but should be installed and started **last**.
Other components are optional, but if they are installed afterwards, Nginx must be restarted (which is not a problem, it's very quick).

Start by creating a Docker network. This line creates one named *platform-net*:
```
docker network create platform-net
```

## Setting up databases

First, run the following command, which sets up necessary databases:
```
docker-compose -p platform.local -f database.yml up -d
```
More specifically, it will set up **MySQL** for AAC, DSS and API Manager, **MongoDB** for Cyclotron and **Postgres** as the DB where data will be saved by NiFi and accessed by DSS.

## AAC

Rename (or make a copy of) `aac.env.example` to `aac.env`. This is now the configuration file for AAC. While we do not need to edit this file to test the tool, you can open it with a text editor to see its configuration. The names of the parameters are fairly self-explanatory.

The `aac-conf/config.yaml` file contains configuration for the apps that will automatically be created for the other components. Again, you do not have to edit this file. By default, each app will be registered with the following credentials:
- **Client ID**: *<SHORT_NAME>_CLIENT_ID*
- **Client secret**: *<SHORT_NAME>_CLIENT_SECRET*

*<SHORT_NAME>* will be *DSS*, *APIM*, *NIFI* or *CYCLOTRON* depending on the component the app is for. Keep in mind that, if you decide to change these credentials, the components' own configuration will have to be changed when you decide to launch them, as they use these values by default.

Run the following, which installs and starts AAC:
```
docker-compose -p platform.local -f aac.yml up -d
```

Shortly after the command returns, AAC should be up.
If an error occurred, it's likely that database setup, which could take a couple minutes, is not yet complete. You can just remove the exited AAC container and repeat the command above.

AAC will create an admin account with credentials `admin`/`admin`.

We are done with AAC. Follow the configuration for the other components you intend to install and finally the Nginx section.

## WSO2 API Manager

Rename `apim.env.example` to `apim.env` and `apim-analytics.env.example` to `apim-analytics.env`. If you didn't change `aac-conf/config.yaml` and are using default certificates, you do not need to edit these files.

Run this to install and start APIM Analytics, which adds features and APIs for analytics to API Manager:
```
docker-compose -p platform.local -f apim-analytics.yml up -d
```

Run this to install and start API Manager:
```
docker-compose -p platform.local -f apim.yml up -d
```

## WSO2 Data Services Server

Rename `dss.env.example` to `dss.env`. Again, if you didn't change `aac-conf/config.yaml` and are using default certificates, you do not need to edit this file.

Run this to install and start DSS:
```
docker-compose -p platform.local -f dss.yml up -d
```

## Apache NiFi

Rename `nifi_param.env.example` to `nifi_param.env`.

Run this to install and start NiFi:
```
docker-compose -p platform.local -f nifi.yml up -d
```

## Cyclotron

Cyclotron has two configuration files, `cyclotron-conf/config.js` (which is where trusted Certificate Authorities are listed) and `cyclotron-conf7configService.js` (which is where client ID and secret are listed). You do not need to edit them, assuming you didn't alter `aac-conf/config.yaml` and are using default certificates.

Run this to install and start Cyclotron:
```
docker-compose -p platform.local -f cyclotron.yml up -d
```

## Nginx

Nginx's configuration file is `nginx.conf`. It contains configurations for each component. If a component is running, you need to uncomment its correspondent section. Conversely, if it is not running, you need to comment it.

Lines are commented by adding a `#` symbol before it. If you're using **Notepad++** to edit the configuration, set *Language > S > Shell* and then you can comment multiple lines at once by selecting them and pressing *Ctrl+K*, or uncomment them with *Ctrl+Shift+K*.

Every component you enabled must be inserted in the **hosts** file of your OS.\
On **Linux**, it is located at:
```
/etc/hosts
```
On **Windows**, it is located at:
```
C:\Windows\System32\drivers\etc\hosts
```

Simply add a line at the bottom of the file, starting with `127.0.0.1` and followed by all the components' domains, separated by whitespaces. For example:
```
127.0.0.1 aac.platform.local api.platform.local gw.platform.local dss.platform.local nifi.platform.local  cyclotron.platform.local
```

You do not need to remove components from *hosts* if you disable them.

Launch Nginx with the following command:
```
docker-compose -p platform.local -f nginx.yml up -d
```

Each component should now be available at the domain chosen for it. Defaults are:
- `aac.platform.local/aac/login` - *AAC*
- `dss.platform.local` - *DSS*
- `api.platform.local` - *API Manager*
- `nifi.platform.local/nifi` - *NiFi*
- `cyclotron.platform.local` - *Cyclotron*
