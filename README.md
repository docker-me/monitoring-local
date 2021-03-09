# monitoring-local

## Used Global Environments
FILE: _.env_
Please following variable update before docker-compose project will be started.
* `DKR_DATA_DIR=/full/path/of/the/repository`

## Usage
FULL/FIRST CONTAINER START
```
$ cd /full/path/of/the/repository
$ docker-compose up -d
OR
$ docker-compose --env-file /full/path/of/the/file/.env --file /full/path/of/the/file/docker-compose.yml up -d
```

REMOVE THE CONTAINER
```
$ cd /full/path/of/the/repository
$ docker-compose down
OR
$ docker-compose --env-file /full/path/of/the/file/.env --file /full/path/of/the/file/docker-compose.yml down
```

CONTAINER START, STOP or RESTART
```
$ cd /full/path/of/the/repository
$ docker-compose [start|stop|restart]
OR
$ docker-compose --env-file /full/path/of/the/file/.env --file /full/path/of/the/file/docker-compose.yml [start|stop|restart]
```
### Used Versions
This docker-compose is used/tested only with UNIX operation systems. 

```
$ docker version
Client: Docker Engine - Community
 Version:           20.10.3
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        48d30b5
 Built:             Fri Jan 29 14:33:21 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.3
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       46229ca
  Built:            Fri Jan 29 14:31:32 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc:
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
```
$ docker-compose version
docker-compose version 1.28.2, build 67630359
docker-py version: 4.4.1
CPython version: 3.7.9
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```

## Components
* traefik
* cadvisor
* prometheus
* pushgateway
* alertmanager
* grafana

### URLs
#### Intern
Below are the used urls between the container:
* http://cadvisor:8080
* http://prometheus:9090
* http://pushgateway:9091
* http://alertmanager:9093
* http://grafana:3000
#### Extern
We can open the application im browser with following URLs:
* http://cadvisor.localhost
* http://prometheus.localhost
* http://pushgateway.localhost
* http://alertmanager.localhost
* http://grafana.localhost   
This URLs are created from traefik.

## traefik
### commands by start
By default, the level is set to ERROR. Alternative logging levels are   
DEBUG, PANIC, FATAL, ERROR, WARN, and INFO.  
* `--log.level=DEBUG`  

Enable the API in insecure mode, which means that the API will be available directly on the entryPoint named traefik.  
* `--api.insecure=true`  

Enabling docker provider:  
* `--providers.docker=true`  

Do not expose containers unless explicitly told so:  
* `--providers.docker.exposedbydefault=false`  

The list of middlewares that are prepended by default to the list of middlewares of each router associated to the named entry point.  
* `--entrypoints.web.address=:80`  


### by the container (labels:)
Explicitly tell Traefik to expose this container  
* `traefik.enable=true`  

The domain the service will respond to  
* ``traefik.http.routers.<container_name>.rule=Host(`<container_name>.localhost`)``  

The port of the container (external)
* `traefik.port=<container_port>`

Allow request only from the predefined entry point named "web"  
* `traefik.http.routers.whoami.entrypoints=web`  

## cadvisor
* https://prometheus.io/docs/guides/cadvisor/
* https://github.com/google/cadvisor
* https://hub.docker.com/r/google/cadvisor/

## prometheus
### config file
* `./prometheus/prometheus.yml`  

### alert rules file
* `./prometheus/alert.rules.yml`  

### commands by start
One of: `debug`, `info`, `warn`, `error`. Only prints messages with levels higher than that.
* `--log.level=info`

To specify which configuration file to load  
* `--config.file`  

Where Prometheus writes its database. Defaults to data/.  
* `--storage.tsdb.path`  

When to remove old data. Defaults to 15d.  
* `--storage.tsdb.retention.time`  

Consoles are exposed on `/consoles/`, and sourced from the directory pointed to by the following flag.  
* `--web.console.templates`

Consoles also have access to all the templates defined with `{{define "templateName"}}...{{end}}` found in `*.lib` files in the directory pointed to by the following flag.  
* `--web.console.libraries`  

The following flag controls access to the administrative HTTP API, which includes functionality such as wiping all the existing metric groups. This is disabled by default. If enabled, administrative functionality will be accessible under the `/api/*/admin/` paths. This flag is important to use pushgateway.  
* `--web.enable-admin-api`   

Prometheus can reload its configuration at runtime. If the new configuration is not well-formed, the changes will not be applied. A configuration reload is triggered by sending a SIGHUP to the Prometheus process or sending a `HTTP POST` request to the `/-/reload` endpoint (when the following flag is enabled). This will also reload any configured rule files.  
* `--web.enable-lifecycle`

## pushgateway
* https://github.com/prometheus/pushgateway

One of: `debug`, `info`, `warn`, `error`. Only prints messages with levels higher than that.
* `--log.level=info`

If specified then enables the Admin API. It lets you perform certain destructive actions. More on that in the following sections.
* `--web.enable-admin-api`

If specified then lets you shutdown the Pushgateway via the API.
* `--web.enable-lifecycle`

### example to PUSH data
Push a single sample into the group identified by `{job="some_job"}`.
Since no type information has been provided, `some_metric` will be of type untyped.
```
echo "some_metric 3.14" | curl --data-binary @- http://pushgateway.localhost/metrics/job/some_job
```

Push something more complex into the group identified by `{job="some_job",instance="some_instance"}`.
Note how type information and help strings are provided. Those lines are optional, but strongly encouraged for anything more complex.
```
cat <<EOF | curl --data-binary @- http://pushgateway.localhost/metrics/job/some_job/instance/some_instance
# TYPE some_metric counter
some_metric{label="val1"} 42
# TYPE another_metric gauge
# HELP another_metric Just an example.
another_metric 2398.283
EOF
```
ERROR
```
pushed metrics are invalid or inconsistent with existing metrics: collected metric "some_metric" { label:<name:"instance" value:"some_instance" > label:<name:"job" value:"some_job" > label:<name:"label" value:"val1" > counter:<value:42 > } is not a UNTYPED
```

## alertmanager
### config file
* `./prometheus/alertmanager.yml`

### commands by start
* `--config.file`  
* `--storage.path`  


## grafana
### environmets
Password and User to use Grafana as administrator
* GF_SECURITY_ADMIN_USER  
* GF_SECURITY_ADMIN_PASSWORD  

## exporter
### blackbox
* https://github.com/prometheus/blackbox_exporter
* https://hub.docker.com/r/prom/blackbox-exporter
* https://github.com/prometheus/blackbox_exporter/blob/master/example.yml

Blackbox Exporter by Prometheus is used to probe endpoints like HTTPS, HTTP, TCP, DNS, and ICMP. After you define the endpoint, the Blackbox exporter generates hundreds of metrics that can be visualized using Grafana. Measuring response time is the most important feature of the Blackbox exporter.

Blackbox Exporter is a self-hosted solution. If you’re looking for something similar, but as SaaS or cloud-based, then you can try Grafana worldPing.

Blackbox exporter configuration file.
* --config.file="blackbox.yml"

The address to listen on for HTTP requests.
* --web.listen-address=":9115"

If true validate the config file and then exit.
* --config.check=true

Only log messages with the given severity or above. One of: [debug, info, warn, error]
* --log.level=info

Output format of log messages. One of: [logfmt, json]
* --log.format=logfmt 

#### httpd
* https://hub.docker.com/_/httpd

#### ngix
* https://hub.docker.com/_/nginx

