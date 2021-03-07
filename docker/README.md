# ARouteServer Docker image

This Docker image can be used to run [ARouteServer](https://github.com/pierky/arouteserver) as a container, either to experiment with it or to easily generate production level configurations that are based on best practices and suggested settings.

## Usage

### Generating configurations

This is a very easy and quick way to obtain secure and feature-rich configurations for Internet Exchange route-servers, based on top of industry best practices and routing security principles.

To generate BIRD or OpenBGPD configurations, route-server clients need to be defined in your local system, in YAML, following the [ARouteServer format](https://github.com/pierky/arouteserver/blob/master/config.d/clients.yml).

The clients need to be stored in a local file, `~/clients.yml` for example.

```yaml
clients:
  - asn: 3333
    ip: "192.0.2.11"
  - asn: 10745
    ip:
    - "192.0.2.22"
    - "2001:db8:1:1::22"
```

It's also possible to build the `clients.yml` file automatically, starting from an existing Euro-IX JSON file, an IXP Manager export file or using records from PeeringDB. For more details, please see [Automatic `clients.yml` creation](https://arouteserver.readthedocs.io/en/latest/USAGE.html#automatic-clients-yml-creation).

Docker can then be instructed to run the image using some input arguments that are needed to specify the route server's ASN, its router ID, which are the local prefixes of the peering LAN, and what's the target BGP daemon and its version (BIRD vs OpenBGPD).

When BIRD 1.x is the target BGP daemon for which the configurations are built, also the `IP_VER` variable must be set, to specify for which address-family the configuration must be generated.

The local file where clients are defined must be mounted into the `/root/clients.txt` path of the container, and the local directory where configuration files need to be saved must be mounted as `/root/arouteserver_configs`.

Example:

```bash
# This is the local directory where configs will be stored:
mkdir ~/arouteserver_configs

docker run \
    -it \
    --rm \
    -v ~/clients.yml:/root/clients.txt:ro \
    -v ~/arouteserver_configs:/root/arouteserver_configs \
    -e RS_ASN=65500 \
    -e ROUTER_ID=192.0.2.123 \
    -e LOCAL_PREFIXES=192.0.2.0/24,2001:db8::/32 \
    -e IP_VER=4 \
    -e DAEMON=bird \
    -e VERSION=1.6.8 \
    pierky/arouteserver:latest
```

After running the container, the configurations will be saved inside the local host's directory, `~/arouteserver_configs` in the example.

Please note: the container will just terminate once the configuration is built; this is the expected behaviour, it's not supposed to stay up, but to only generate the config and then shutdown.

#### Customizing the route-server features

The options used to build the configuration files can be customized by providing a user-defined general.yml file at run-time.

This can be done by mounting the local host's general.yml file into the `/etc/arouteserver/general.yml` path of the container.

Details on all the available features and options can be found in the [doc site](https://arouteserver.readthedocs.io/en/latest/CONFIG.html) or in the [commented version of general.yml](https://github.com/pierky/arouteserver/blob/master/config.d/general.yml).

```bash
docker run \
    -it \
    --rm \
    -v ~/custom-general.yml:/etc/arouteserver/general.yml:ro \
    ... # other options
```

### Interactive execution

To experiment with the tool, the Docker image can be executed interactively:

```bash
docker run \
    -it \
    --rm \
    pierky/arouteserver:latest bash
```

This will give you access to an environment where ARouteServer and its dependencies (bgpq4) are installed and ready to be used.

## Dockerfile

The Dockerfile can also provide some useful hints on how to install and configure ARouteServer on a host that is different than a Docker image. You can [check it out on GitHub](https://github.com/pierky/arouteserver/blob/master/docker/Dockerfile).

## Tag `latest`

The image is built on top of the latest stable release of ARouteServer. Other tags are also available, please visit the Tags section on Docker Hub.

# Author

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)

