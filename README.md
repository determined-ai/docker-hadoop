
This repository contains docker images for basic hadoop cluster.
## Configuration loading

The containers supports multiple configuration loading mechanism. (All of the configuration loading is defined in the [base-docker](https://github.com/elek/docker-bigdata-base) image. The configuration methods are stored in the `/opt/configurer`  directory and could be selected by setting the environment variable `CONFIG_TYPE`

You can see various example configuration, ansible and docker-compose scripts at the main [umbrella repository](https://github.com/elek/bigdata-docker).

The three main configuration loading mechanis is:

 * ```CONFIG_TYPE=simple```: Using some simple default and configuration defined with environment variables.
 * ```CONFIG_TYPE=consul```: Using configuration files (and not list of ```key: value``` pairs) stored in a consul. Supports dynamic restart if the configuration is changing.

### Simple configuration

This is the default configuration.

Every configuration file is defined with a list of ```key: value``` pairs, even if they will be converted finally to hadoop xml format.

The destination format is defined by the extensions (or by an additional format specifier)

The generated files will be saved to the `$CONF_DIR` directory.

#### Naming convention for set config keys from enviroment variables

To set any configuration variable you shold follow the following pattern:

```
NAME.EXTENSION_configkey=VALUE
```

The extension could be any extension which has a predefined transformation (currently xml, yaml, properties, configuration, yaml, env, sh, conf, cfg)

examples:

```
CORE-SITE_fs.default.name: "hdfs://localhost:9000"
HDFS-SITE_dfs_namenode_rpc-address: "localhost:9000"
HBASE-SITE.XML_hbase_zookeeper_quorum: "localhost"
```

In some rare cases the transformation and the extension should be different. For example the kafka `server.properties` should be in the format `key=value` which is the `cfg` transformation in our system. In that case you can postfix the extension with an additional format specifier:


```
NAME.EXTENSION!FORMAT_configkey=VALUE
```

For example:

```
SERVER.CONF!CFG_zookeeper.address=zookeeper:2181
```

#### Available transformation

 * xml: HADOOP xml file format
 * properties: key value pairs with ```:``` as separator
 * cfg: key value pairs with ```=``` as separator
 * conf: key value pairs with space as spearator (spark-defaults is an example)
 * env: key value paris with ```=``` as separator
 * sh: as the env but also includes the export keyword

#### Example

The simple directory in the [bigdata-docker](https://github.com/elek/bigdata-docker) project contains a [docker-compose](https://github.com/elek/bigdata-docker/blob/master/simple/docker-compose.yaml) example using simple configuration loading.

### Consul config loading

Could be activated with ```CONFIG_TYPE=consul```

* The starter script list the configuration file names based on a consul key prefix. All the files will be downloaded from the consul key value store and the application process will be started with consul-template (enable an automatic restart in case of configuration file change)

The source code of the consul based configuration loading and launcher is available at the [elek/consul-launcher](https://github.com/elek/consul-launcher) repository.

#### Configuration

 * `CONSUL_PATH` defines the root of the subtree where the configuration are downloaded from. The root could also contain a configuration `config.ini`. Default is `conf`

 *  `CONSUL_KEY` is optional. It defines a subdirectory to download the the config files. If both `CONSUL_PATH` and `CONSUL_KEY` are defined, the config files will be downloaded from `$CONSUL_PATH/$CONSUL_KEY` but the config file will be read from `$CONSUL_PATH/config.ini`

## Examples

For getting started use the incloded docker-compose file and start both hdfs and yarn clusters:

```
docker-compose up -d
```

You can adjust the settings in the compose-config file.

To scale up datanode/namenode:

```
docker-compose scale datanode=3
```

To check namenode/resourcemanager use the published ports:

* Resourcemanager: http://localhost:8080
* Namenode: http://localhost:50070 (in case of hadoop 2.x)

## Smoketest

```
docker-compose exec resourcemanager /opt/hadoop/bin/yarn jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar pi 16 1000
```
## Versioning policy

  The _latest_ tag points to the latest configuration loading and the latest stable apache version.

  The _testing_ usually points to a cutting edge developer snapshot or RC/alpha release but expected to be working. 

  If there is plain version tag without prefix it is synchronized with the version of the original apache software.

  It there is a prefix (eg. HDP) it includes a specific version from a specific distribution.

  As the configuration loading in the base image is constantly evolving even the tags of older releases may be refreshed over the time.

## Local build

Use the included Makefile

```
make build
```

You can adjust the VERSION and URL environment variables.