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
```
docker network create platform-net
```

## AAC

### Installing AAC

First, run the following command, which sets up necessary databases:
```
docker-compose -p platform.local -f database.yml up -d
```

Rename (or make a copy of) `aac.env.example` to `aac.env`. This is now the configuration file for AAC. While we do not need to edit any of the variables contained within to test the tool, you can open the file with a text editor to see its configuration. The names of the parameters are fairly self-explanatory.

If the previous Docker Compose command has returned, run the following, which installs and starts AAC:
```
docker-compose -p platform.local -f aac.yml up -d
```

Shortly after the command returns, AAC should be up. Open `localhost:8080/aac/login` in your browser and you should be prompted to log in. If you didn't change them, credentials should be `admin`/`admin`.

We will later map AAC to a domain, but for now, we need to create applications for other components.

### Creating applications for the components

You should see a ***Client Apps*** page, with only one entry, corresponding to API Manager. Click on it and take not of **clientId** and **clientSecret**.

#### DSS

Return to *Client Apps* and click on ***NEW APP***. Choose a name for it, for example, `dss`.

The details of the application are listed. Take notes of the generated *clientId* and *clientSecret*, then switch to the ***Settings*** tab.

In **redirect Web server URLs**, enter the following (assuming you will be using default domains for the components):
```
https://dss.platform.local
```

Under **Grant types**, check the following:
- `Implicit`
- `Authorization Code`
- `Client Credentials`

Under **Enabled identity providers**, check `internal`. This will need to be approved, which we will do later.

Click ***Save settings***, then switch to the ***API Access*** tab. Scopes are grouped in different services. Check scopes as follows, confirming with ***Save permissions*** for every service you edit:

**Basic profile service**
- openid
- profile.basicprofile.me
- profile
- email
- profile.basicprofile.all
- profile.accountprofile.me

**Role Management Service**
- user.roles.me
- user.roles.read
- user.roles.read.all
- client.roles.read.all

When done, click on ***Admin*** in the top bar, switch to the ***IdP Approvals*** tab and ***Approve*** the app.

You need to create apps for NiFi and Cyclotron as well.

#### NiFi

***Settings***
- Take note of *clientId* and *clientSecret*.
- *redirect Web server URLs*: `https://nifi.platform.local/nifi-api/access/oidc/callback`
- *Grant types*: `Implicit`, `Authorization Code`, `Client Credentials`
- *Enabled identity providers*: `internal` (remember to approve it)

***API Access***
**Basic profile service**
- openid
- profile.basicprofile.me
- profile
- email
- profile.accountprofile.me

**Role Management Service**
- user.roles.me
- user.roles.read
- user.roles.read.all

#### Cyclotron

***Settings***
- Take note of *clientId* and *clientSecret*.
- *redirect Web server URLs*: `https://cyclotron.platform.local`
- *Grant types*: `Implicit`, `Authorization Code`,  `Password`, `Client Credentials`, `Refresh Token`, `Native`
- *Enabled identity providers*: `internal` (remember to approve it)

***API Access***
**Basic profile service**
- openid
- profile.basicprofile.me

**Role Management Service**
- user.roles.me

We are done with AAC. Follow the configuration for the other components you intend to install and finally the Nginx section.

## WSO2 API Manager

APIM Analytics adds features for analyzing API usage in API Manager. It is not required to run API Manager, but this section will guide you through installing it as well.

Rename `apim.env.example` to `apim.env` and open it with a text editor.

Set the correct names and passwords for the files containing the certificates. Defaults are:
```
APIM_KEYSTORE_FILENAME=apigwself.jks
APIM_KEYSTORE_PASS=platform
APIM_KEYSTORE_KEYALIAS=scoapim
APIM_TRUSTSTORE_FILENAME=client-truststore.jks
APIM_TRUSTSTORE_PASS=platform
```

Set client ID and secret to the ones you previously took note of from AAC. Defaults are:
```
AAC_CONSUMERKEY=API_MGT_CLIENT_ID
AAC_CONSUMERSECRET=YOUR_MNGMT_CLIENT_SECRET
```

If you intend to use **APIM Analytics**, set the following parameter as well:
```
ANALYTICS_ENABLED=true
```

Rename `apim-analytics.env.example` to `apim-analytics.env` and open it with a text editor.

Set the correct names and passwords for the files containing the certificates. Defaults are:
```
APIM_KEYSTORE_FILENAME=am-analytics.jks
APIM_KEYSTORE_PASS=platform
APIM_KEYSTORE_KEYALIAS=scoapim
APIM_TRUSTSTORE_FILENAME=client-truststore.jks
APIM_TRUSTSTORE_PASS=platform
```

Run this to install and start APIM Analytics:
```
docker-compose -p platform.local -f apim-analytics.yml up -d
```

Run this to install and start API Manager:
```
docker-compose -p platform.local -f apim.yml up -d
```

## WSO2 Data Services Server

Rename `dss.env.example` to `dss.env` and open it with a text editor.

Set AAC_HOSTNAME to the main page of AAC. Default is:
```
AAC_HOSTNAME=https://aac.platform.local/aac
```

Set the client ID and secret to the values you got from AAC:
```
AAC_CONSUMERKEY=dss-client-id-here
AAC_CONSUMERSECRET=dss-client-secret-here
```

Set port to 443:
```
DSS_PORT="443"
```

Set the correct names and passwords for the files containing the certificates. Defaults are:
```
DSS_KEYSTORE_FILENAME=dss.jks
DSS_KEYSTORE_PASS=platform
DSS_KEYSTORE_KEYALIAS=dss
DSS_TRUSTSTORE_FILENAME=client-truststore.jks
DSS_TRUSTSTORE_PASS=platform
```

Run this to install and start DSS:
```
docker-compose -p platform.local -f dss.yml up -d
```

## Apache NiFi

Rename `nifi_param.env.example` to `nifi_param.env` and open it with a text editor.

Set the correct names and passwords for the files containing the certificates. Defaults are:
```
KEYSTORE_PATH=/opt/nifi/nifi-current/certs/keystore.jks
KEYSTORE_TYPE=JKS
KEYSTORE_PASSWORD=platform
TRUSTSTORE_PATH=/opt/nifi/nifi-current/certs/truststore.jks
TRUSTSTORE_PASSWORD=platform
TRUSTSTORE_TYPE=JKS
OIDC_PROVIDER_TRUSTSTORE_PATH=/opt/nifi/nifi-current/certs/truststore.jks
OIDC_PROVIDER_TRUSTSTORE_PASSWD=platform
```

Set the client ID and secret to the values you got from AAC:
```
NIFI_SECURITY_USER_OIDC_CLIENT_ID=nifi-client-id-here
NIFI_SECURITY_USER_OIDC_CLIENT_SECRET=nifi-client-secret-here
```

Run this to install and start NiFi:
```
docker-compose -p platform.local -f nifi.yml up -d
```

## Cyclotron

Cyclotron has two configuration files, `config.js` and `configService.js`. Both are found in the *cyclotron-conf* subfolder.

Open *configService.js* with a text editor and look for a line containing clientID. Change its value to the value you got from AAC.
```
      clientID: 'cyclotron-client-id-here',
```

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

You do not need to remove components if you disable them. Keep in mind that deploying the platform on Windows is not fully supported, due to how Docker for Windows uses the hosts file.

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
