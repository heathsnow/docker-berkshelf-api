# docker-berkshelf-api

This is the berkshelf-api 3.0.0 on a ruby:2.3-alpine base image.

This work is based on https://github.com/xmik/berkshelf-api-docker/

Main difference is that this is based on Alpine Linux, which results in a much
smaller final image.

## Usage
Assuming that:
  * your Chef Server is already running and is accessible by chef.example.com
  * there is Chef Server user: berkshelf (probably client is also ok, but
   never both, you'll most probably get Permission denied when authorizing as
   berkshelf)

Run this docker image e.g. like this:
```bash
$ docker run -dti --name berks -v ${PWD}/test/berkshelf.pem:/home/berkshelf/.chef/berkshelf.pem -e CHEF_SERVER_ENDPOINT="https://chef.example.com:443" -e BERKS_BUILD_INTERVAL=15 -e CHEF_ORGANIZATION="/organizations/testorg" --link chef_server:chef.example.com -p 26200:26200  hrak/berkshelf-api-docker:0.0.4
```

where:
  * you have to mount pem file for Chef Server user: berkshelf
  * `CHEF_SERVER_ENDPOINT` is your Chef Server url reachable from chef workstations.
   Default is "https://chef.example.com:443". It should be **full Chef Server
   protocol and domain name and port**.
  * `CHEF_CLIENT_NAME` is the Chef Server user or client name, default is "berkshelf"
  * `BERKS_BUILD_INTERVAL` sets build_interval - the number of seconds before it
   refreshes from the endpoints (see [berkshelf/berkshelf-api](https://github.com/berkshelf/berkshelf-api)),
   default here is 5, just as in berkshelf-api. berkshelf-api, if correctly
   configured, sends 9 messages per build_interval.
  * `CHEF_ORGANIZATION` is obligatory if you use Chef Server 12. Default is empty,
   so that it works for Chef Server 11. Example value is: "/organizations/testorg"
   and it must start with slash.
  * the link is necessary only if `CHEF_SERVER_ENDPOINT` is not reachable for
   berks container, so e.g. when testing.

When berks container is already running and you want to change some configuration,
 editing `/home/berkshelf/.berkshelf/api-server/config.json` file will have no
 effect, because it is generated. Edit then this file: `/usr/bin/run_berks_api.sh`
 and restart the container. To login into the container:
```bash
$ docker exec -ti berks /bin/bash
```

### Verification
To confirm that you have connected from chef workstation to berks-api server,
 search for some cookbook in your Chef Server, e.g. apt:
```bash
$ berks search apt --source http://[container_ip]:26200
apt (2.6.1)
```

