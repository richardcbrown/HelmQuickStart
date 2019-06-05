# Helm Quick Start Guide

## A guide to setting up Helm locally

# Key components

The following set of projects will need to be pulled from github:

## PulseTile

PulseTile is currently available at:
https://github.com/richardcbrown/PulseTile-RA/tree/release/1.0.0
And serves as the front end of the Helm project.

## Qewd-Courier

Qewd-Courier is currently available at:
https://github.com/richardcbrown/QEWD-Courier/tree/release/1.0.0
Qewd-Courier provides several core microservices to the middleware of Helm.
The microservices we currently use from Qewd-Courier are:
- initialisation_service - this service allows for coordination of tasks that need to be performed by other microservices when a user first logs into Helm.
- openehr_service - provides connectivity to EtherCIS for the Qewd middleware.
- orchestrator - service that provides overarching coordination of Qewd services.

## Qewd System of Systems integration service

The System of System integration service is currently available at:
https://github.com/richardcbrown/QC-MPI-Microservice/tree/release/1.0.0
This service provides connectivity between the Qewd middleware and the system of systems.

## Qewd Oidc Client

The oidc client is currently available at:
https://github.com/LeedsCC/oidc-client
The oidc client delegates authentication to NHS login and handles the setup of tokens for users within Qewd.

## Qewd Oidc Provider

The oidc provider is currently available at:
https://github.com/LeedsCC/oidc-provider
The oidc provider service is used to allow pulling of Helm data by healthcare trusts. It is also used locally to mock NHS login functionality.

## EtherCIS

Ether CIS is currently available at:
https://github.com/ethercis/deploy-n-scripts
The service provides connectivity to the postgres data store, storing information in the OpenEHR format.

You will also need to pull:
https://github.com/ethercis/ehrservice
As we will need the migration scripts.

## PostgreSQL

EtherCIS uses PostgreSQL 10 as the data store, it can be downloaded from:
https://www.enterprisedb.com/downloads/postgres-postgresql-downloads
10.8 is recommended

## YottaDB Files

The base files to setup yottaDB can be found from:
https://github.com/robtweed/yotta-gbldir-files

# Local Installation

## PostgreSQL setup

You will need maven installed to perform the database migrations (and JAVA).

https://maven.apache.org/guides/getting-started/windows-prerequisites.html

Once PostgreSQL has been installed two further extension are required to allow EtherCIS to work:
https://github.com/arkhipov/temporal_tables
https://github.com/postgrespro/jsquery

If you are installing on Windows, the extensions are available from:
https://github.com/arkhipov/temporal_tables/releases/tag/v1.2.0

and:
https://www.postgresonline.com/journal/archives/375-PostgreSQL-JSQuery-extension-Windows-binaries.html

download and extract the files to your machine, the files in the lib and share/extension folders should be copied into the relevant PostgreSQL location (e.g C:\Program Files\PostgreSQL\10\lib and C:\Program Files\PostgreSQL\10\share\extension)

With the extensions installed you should now execute the database creation scripts.

These can be found https://github.com/ethercis/ehrservice/blob/remote-github/db/createdb.sql

and https://github.com/ethercis/ehrservice/blob/remote-github/ecisdb/createdb.sql

You can use the SQL Shell that installs as part of the PostgreSQL install to do this.

Once this is complete we are nearly ready to start up EtherCIS.

The final things to do are:

make sure EtherCIS knows the location of your PostgreSQL server:

In:

```
~\EtherCIS\deploy-n-scripts\ethercis-install\v1.3.0\config\services.properties
```

Make sure the ```server.persistence.jooq.url``` property is set to your PostgreSQL instance.

In here you can also set ```server.http.port``` to whichever port you want EtherCIS to listen on.

In:

```
~\EtherCIS\ehrservice\ecisdb\pom.xml
```

Set: 

```        
<database.url>jdbc:postgresql://127.0.0.1:5434/ethercis/ehr</database.url>
```

To the location of your PostgreSQL instance.

Then in:

```
~\EtherCIS\ehrservice\ecisdb
```
Run:

```
mvn compile
mvn flyway:migrate
```

In 

```
~\EtherCIS\deploy-n-scripts\ethercis-install\v1.3.0\config\knowledge
```

Make sure the following folders exist (they may be empty):
```
archetypes
templates
```

In this directory there should also be a folder called:

```
operational_templates
```

Copy the ```IDCR - Top3issues.opt``` file provided in this repository into this folder.

This should now have setup all the relevant PostgreSQL components for EtherCIS.

To start up EtherCIS:

```
java -jar lib/ethercis-1.3.0-SNAPSHOT-runtime.jar -propertyFile config/services.properties
```

From:

```
~\EtherCIS\deploy-n-scripts\ethercis-install\v1.3.0
```

## Hosts configuration

The following setup assumes that you have a host override configured in your machine hosts file of:

```
127.0.0.1   helm-local.com
```

However you can setup the hostname to be whatever you want.

## PulseTile

A set of nginx configuration files for the PulseTile docker container are provided in this repository (See PulseTileFiles), these should be copied to the root folder of the PulseTile repository you pulled from github (you may need to update the proxy_pass host address in default.conf).

In order to create a build of the PulseTile ui we need to setup the domain for PulseTile:

In:

```
~\PulseTile-RA\public\index.html
```

Find:

```
<script type="text/javascript">
    window.config = {
        domainName: "",
        lightPalette: {
            mainColor: "#3596f4",
            dangerColor: "#da534f",
            contrastColor: "#000",
            disabledColor: "#e9e4e4",
            borderColor: "#e5e5e5",
            paperColor: "#fff",
            toolbarColor: "#e5e5e5",
            fontColor: "#000",
            viewButton: "#30ad57",
        },
        darkPalette: {
            mainColor: "#000",
            dangerColor: "#da534f",
            contrastColor: "#000",
            disabledColor: "#e9e4e4",
            borderColor: "#000",
            paperColor: "#fff",
            toolbarColor: "#fff",
            fontColor: "#000",
            viewButton: "#000",
        },
    }
</script>
```

And configure the domain name property (in this case it should be http://helm-local.com).

Then from:

```
~\PulseTile-RA
```

Run:

```
npm run build
```

This should compile the PulseTile project, you should see files in the ```~\PulseTile-RA\build``` folder.

This should be all the setup required for PulseTile.

## Qewd

The docker-compose file provided in this repository can be used to bring all microservices online.
You will need to set the correct volume paths so they point to where you downloaded the services to.

You will need to place the correct yottaDB files in the correct locations (the \yottaDB directory) for the following services:

- orchestrator
- openehr_service
- fhir_service

The following configuration files will need to be updated:

## fhir_service

Configuration file is located at:

```
~\QC-MPI-Microservice\configuration\fhir_service.config.json
```

The following properties will need to be setup to match the test YHCR server:

```
auth.host
auth.grant_type
auth.client_id
auth.client_secret
api.host
```

If you are using grant_type "urn:ietf:params:oauth:grant-type:jwt-bearer" you will also need to place your private key file (name privateKey.key) in the same folder as this configuration file.

## oidc client

Copy the test configuration files in OidcClientFiles to:
```
~\oidc-client\configuration
```

## oidc provider

Copy the test configuration files in OidcProviderFiles to:
```
~\oidc-provider\configuration
```

## Qewd-Courier

Copy the test configuration files in QewdCourierFiles to:
```
~\QEWD-Courier\helm\configuration
```

You will need to set the following properties in:

```
~\QEWD-Courier\helm\configuration\global_config.json

openehr.servers.ethercis.username
openehr.servers.ethercis.password
```

These credentials should match the credentials setup in the EtherCIS file:

```
~\EtherCIS\deploy-n-scripts\ethercis-install\v1.3.0\config\security\authenticate.ini
```

You may have to update the host names in the supplied setup files if you chose a different host name for your local site.

# Startup

When first starting up the system, you will need to start the orchestrator qewd service first, this allows it to install shared node_modules for the Qewd-Courier portion.
The volumes will need to be updated to match your install locations.

```
docker run -it --name orchestrator --rm --net helmquickstart_qewd-net -p 8080:8080 -v ~/helm:/opt/qewd/mapped -v ~/yottadb/orchestrator:/root/.yottadb/r1.24_x86_64/g rtweed/qewd-server
```

Once the the orchestrator has started and installed node modules you may stop it again. We will use docker compose to bring the Helm services online.

With EtherCIS started, you should be able to use the docker-compose file supplied with this repository:

```
docker-compose up
```

On first run some services will import their routes into the orchestrator and will then stop.
Just run:

```
docker-compose up
```

Again to bring the services back up.

This should bring your local version of Helm online.
You can check by going to http://helm-local.com