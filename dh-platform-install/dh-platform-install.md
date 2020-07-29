# Installing the Digital Hub components with Docker Compose

This document will guide you through the configuration and installation of the components of the *Digital Hub Platform*.

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

```docker network create platform-net```

## AAC

Rename (or make a copy of) `aac.env.example` to `aac.env`. This is now the configuration file for AAC. Open it with a text editor.

## WSO2 API Manager

Rename `apim.env.example` to `apim.env`. This is now the configuration file for API Manager.

## WSO2 Data Services Server

Rename `dss.env.example` to `dss.env`. This is now the configuration file for Data Services Server (DSS).

## Apache NiFi

Rename `nifi_param.env.example` to `nifi_param.env`. This is now the configuration file for NiFi.

## Cyclotron

Cyclotron has two configuration files, `config.js` and `configService.js`. Both are found in the *cyclotron-conf* subfolder.

## Nginx

Nginx's configuration file is `nginx.conf`. It contains configurations for each component. If a component is running, you need to uncomment its correspondent section. Conversely, if it is not running, you need to comment it.

Lines are commented by adding a `#` symbol before it. If you're using **Notepad++** to edit the configuration, set *Language > S > Shell* and then you can comment multiple lines at once by selecting them and pressing *Ctrl+K*, or uncomment them with *Ctrl+Shift+K*.

Every component you enabled must be inserted in the **hosts** file of your OS.\
On **Linux**, it is located at:\
```/etc/hosts```\
On **Windows**, it is located at:\
```C:\Windows\System32\drivers\etc\hosts```

Simply add a line at the bottom of the file, starting with `127.0.0.1` and followed by all the components' domains, separated by whitespaces. For example:

```127.0.0.1 aac.platform.local api.platform.local gw.platform.local dss.platform.local nifi.platform.local  cyclotron.platform.local```

You do not need to remove components if you disable them. Keep in mind that deploying the platform on Windows is not fully supported, due to how Docker for Windows uses the hosts file.
