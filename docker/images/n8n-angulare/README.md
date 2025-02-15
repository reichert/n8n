# n8n - Workflow Automation

![n8n.io - Workflow Automation](https://raw.githubusercontent.com/n8n-io/n8n/master/assets/n8n-logo.png)

n8n is a free and open [fair-code](http://faircode.io) distributed node based Workflow Automation Tool. It can be self-hosted, easily extended, and so also used with internal tools.

<a href="https://raw.githubusercontent.com/n8n-io/n8n/master/assets/n8n-screenshot.png"><img src="https://raw.githubusercontent.com/n8n-io/n8n/master/assets/n8n-screenshot.png" width="550" alt="n8n.io - Screenshot"></a>

## Contents

  - [Demo](#demo)
  - [Available integrations](#available-integrations)
  - [Documentation](#documentation)
  - [Start n8n in Docker](#start-n8n-in-docker)
  - [Start with tunnel](#start-with-tunnel)
  - [Securing n8n](#securing-n8n)
  - [Persist data](#persist-data)
  - [Passing Sensitive Data via File](#passing-sensitive-data-via-file)
  - [Updating a Running docker-compose Instance](#updating-a-running-docker-compose-instance)
  - [Example Setup with Lets Encrypt](#example-setup-with-lets-encrypt)
  - [What does n8n mean and how do you pronounce it](#what-does-n8n-mean-and-how-do-you-pronounce-it)
  - [Support](#support)
  - [Jobs](#jobs)
  - [Upgrading](#upgrading)
  - [License](#license)

## Demo

[:tv: A short demo (< 3 min)](https://www.youtube.com/watch?v=3w7xIMKLVAg)
which shows how to create a simple workflow which automatically sends a new
Slack notification every time a Github repository received or lost a star.


## Available integrations

n8n has 200+ different nodes to automate workflows. The list can be found on: [https://n8n.io/nodes](https://n8n.io/nodes)


## Documentation

The official n8n documentation can be found under: [https://docs.n8n.io](https://docs.n8n.io)

Additional information and example workflows on the n8n.io website: [https://n8n.io](https://n8n.io)


## Start n8n in Docker

```
docker run -it --rm \
	--name n8n \
	-p 5678:5678 \
	-v ~/.n8n:/home/node/.n8n \
	n8nio/n8n
```

You can then access n8n by opening:
[http://localhost:5678](http://localhost:5678)


## Start with tunnel

> **WARNING**: This is only meant for local development and testing. Should not be used in production!

To be able to use webhooks which all triggers of external services like Github
rely on n8n has to be reachable from the web. To make that easy n8n has a
special tunnel service (uses this code: [https://github.com/localtunnel/localtunnel](https://github.com/localtunnel/localtunnel)) which redirects requests from our servers to your local
n8n instance.

To use it simply start n8n with `--tunnel`

```
docker run -it --rm \
	--name n8n \
	-p 5678:5678 \
	-v ~/.n8n:/home/node/.n8n \
	n8nio/n8n \
	n8n start --tunnel
```

## Securing n8n

By default n8n can be accessed by everybody. This is OK if you have it only running
locally but if you deploy it on a server which is accessible from the web you have
to make sure that n8n is protected!
Right now we have very basic protection via basic-auth in place. It can be activated
by setting the following environment variables:

```
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=<USER>
N8N_BASIC_AUTH_PASSWORD=<PASSWORD>
```

## Persist data

The workflow data gets by default saved in an SQLite database in the user
folder (`/home/node/.n8n`). That folder also additionally contains the
settings like webhook URL and encryption key.

```
docker run -it --rm \
	--name n8n \
	-p 5678:5678 \
	-v ~/.n8n:/home/node/.n8n \
	n8nio/n8n
```

### Start with other Database

By default n8n uses SQLite to save credentials, past executions and workflows.
n8n however also supports PostgresDB, MySQL and MariaDB. To use them simply a few
environment variables have to be set.

It is important to still persist the data in the `/root/.n8n` folder. The reason
is that it contains n8n user data. That is the name of the webhook
(in case) the n8n tunnel gets used and even more important the encryption key
for the credentials. If none gets found n8n creates automatically one on
startup. In case credentials are already saved with a different encryption key
it can not be used anymore as encrypting it is not possible anymore.

#### Use with PostgresDB

Replace the following placeholders with the actual data:
  - POSTGRES_DATABASE
  - POSTGRES_HOST
  - POSTGRES_PASSWORD
  - POSTGRES_PORT
  - POSTGRES_USER
  - POSTGRES_SCHEMA

```
docker run -it --rm \
	--name n8n \
	-p 5678:5678 \
	-e DB_TYPE=postgresdb \
	-e DB_POSTGRESDB_DATABASE=<POSTGRES_DATABASE> \
	-e DB_POSTGRESDB_HOST=<POSTGRES_HOST> \
	-e DB_POSTGRESDB_PORT=<POSTGRES_PORT> \
	-e DB_POSTGRESDB_USER=<POSTGRES_USER> \
	-e DB_POSTGRESDB_SCHEMA=<POSTGRES_SCHEMA> \
	-e DB_POSTGRESDB_PASSWORD=<POSTGRES_PASSWORD> \
	-v ~/.n8n:/home/node/.n8n \
	n8nio/n8n \
	n8n start
```

A full working setup with docker-compose can be found [here](https://github.com/n8n-io/n8n/blob/master/docker/compose/withPostgres/README.md)

#### Use with MySQL

Replace the following placeholders with the actual data:
  - MYSQLDB_DATABASE
  - MYSQLDB_HOST
  - MYSQLDB_PASSWORD
  - MYSQLDB_PORT
  - MYSQLDB_USER

```
docker run -it --rm \
	--name n8n \
	-p 5678:5678 \
	-e DB_TYPE=mysqldb \
	-e DB_MYSQLDB_DATABASE=<MYSQLDB_DATABASE> \
	-e DB_MYSQLDB_HOST=<MYSQLDB_HOST> \
	-e DB_MYSQLDB_PORT=<MYSQLDB_PORT> \
	-e DB_MYSQLDB_USER=<MYSQLDB_USER> \
	-e DB_MYSQLDB_PASSWORD=<MYSQLDB_PASSWORD> \
	-v ~/.n8n:/home/node/.n8n \
	n8nio/n8n \
	n8n start
```

## Passing Sensitive Data via File

To avoid passing sensitive information via environment variables "_FILE" may be
appended to some environment variables. It will then load the data from a file
with the given name. That makes it possible to load data easily from
Docker- and Kubernetes-Secrets.

The following environment variables support file input:
  - DB_POSTGRESDB_DATABASE_FILE
  - DB_POSTGRESDB_HOST_FILE
  - DB_POSTGRESDB_PASSWORD_FILE
  - DB_POSTGRESDB_PORT_FILE
  - DB_POSTGRESDB_USER_FILE
  - DB_POSTGRESDB_SCHEMA_FILE
  - N8N_BASIC_AUTH_PASSWORD_FILE
  - N8N_BASIC_AUTH_USER_FILE

## Example Setup with Lets Encrypt

A basic step by step example setup of n8n with docker-compose and Lets Encrypt is available on the
[Server Setup](https://docs.n8n.io/#/server-setup) page.

## Updating a running docker-compose instance

```
# Pull down the latest version from dockerhub
docker pull n8nio/n8n
# Stop current setup
sudo docker-compose stop
# Delete it (will only delete the docker-containers, data is stored separately)
sudo docker-compose rm
# Then start it again
sudo docker-compose up -d
```

## Setting Timezone

To define the timezone n8n should use, the environment variable `GENERIC_TIMEZONE` can
be set. This gets used by for example the Cron-Node.
Apart from that can also the timezone of the system be set separately. Which controls what
some scripts and commands return like `$ date`. The system timezone can be set via
the environment variable `TZ`.

Example to use the same timezone for both:
```
docker run -it --rm \
	--name n8n \
	-p 5678:5678 \
	-e GENERIC_TIMEZONE="Europe/Berlin" \
	-e TZ="Europe/Berlin" \
	n8nio/n8n
```


## Build Docker-Image

```
# Subir a versão do n8n para o servidor de npm (npm.angular-ecommerce.com)
## Pré-reqs:
- isso para configurar o npm local e criar usuário no npm da angular (verdaccio)
npm config set @angularecommerce:registry http://npm.angular-ecommerce.com
npm config set always-auth true
npm adduser --registry http://npm.angular-ecommerce.com --scope=@angularecommerce
npm login

# alterar o package.json
git add .
git commit -m "0.126.102"
git push origin master
git tag -a 0.126.102 -m "Release 0.126.102"
git push origin 0.126.102
npm publish

- no packages.json, incluir isso:

- no Dockerfile, alterar o lugar de onde ele vai baixar o n8n via npm install


# Criar a imagem do docker
docker buildx build --platform linux/amd64 --build-arg N8N_VERSION=0.126.100 -t areichert/n8n:0.126.100 .
```


## What does n8n mean and how do you pronounce it?

**Short answer:** It means "nodemation" and it is pronounced as n-eight-n.

**Long answer:** I get that question quite often (more often than I expected)
so I decided it is probably best to answer it here. While looking for a
good name for the project with a free domain I realized very quickly that all the
good ones I could think of were already taken. So, in the end, I chose
nodemation. "node-" in the sense that it uses a Node-View and that it uses
Node.js and "-mation" for "automation" which is what the project is supposed to help with.
However, I did not like how long the name was and I could not imagine writing
something that long every time in the CLI. That is when I then ended up on
"n8n". Sure does not work perfectly but does neither for Kubernetes (k8s) and
did not hear anybody complain there. So I guess it should be ok.



## Support

If you have problems or questions go to our forum, we will then try to help you asap:

[https://community.n8n.io](https://community.n8n.io)




## Jobs

If you are interested in working for n8n and so shape the future of the project
check out our [job posts](https://apply.workable.com/n8n/)




## Upgrading

Before you upgrade to the latest version make sure to check here if there are any breaking changes which concern you:
[Breaking Changes](https://github.com/n8n-io/n8n/blob/master/packages/cli/BREAKING-CHANGES.md)



## License

n8n is [fair-code](http://faircode.io) distributed under [**Apache 2.0 with Commons Clause**](https://github.com/n8n-io/n8n/blob/master/packages/cli/LICENSE.md) license

Additional information about license can be found in the [FAQ](https://docs.n8n.io/#/faq?id=license)
