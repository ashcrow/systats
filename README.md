# talook
[![Build Status](https://api.travis-ci.org/RHInception/talook.png)](https://travis-ci.org/RHInception/talook/)

Single web front end for [jsonstats](https://github.com/RHInception/jsonstats).


## Features
* Only requires Python (w/ simplejson if 2.4 or 2.5)
* Python 2.4+ compatible
* Simple filesystem based caching
* Access and application logging
* JSON based configuration
* Bookmarkable hosts
* host and environment REST endpoints
* Ajaxy
* Unit tested
* Optional reload of standalone server on configuration update


## Unittests
Use *./setup.py test* from the main directory to execute unittests.

## Configuration
Configuration of the server is done in JSON and is by default kept in the current directories config.json file.
You can override the location by setting `TALOOK_CONFIG_FILE` environment variable or using the `-c`/`--config`
switch on the all in one server.

| Name          | Type | Required | Value                                         |
|---------------|------|----------|-----------------------------------------------|
| cachedir      | str  | *False*  | Full path to the cache directory. If this is empty the cache is disabled |
| cachetime     | dict | *False*  | kwargs for Python's datetime.timedelta [1](http://docs.python.org/2.6/library/datetime.html#datetime.timedelta) |
| endpoint      | str  | *True*   | Endpoint url to pull json data from with a `%s` placeholder for hostname   |
| extranotes    | str  | *False*  | URL of external page with more info about a host with a `%s` placeholder for hostname |
| hosts         | dict | *True*   | hostname: environment pairs                   |
| logdir        | str  | *True*   | Full path to the log directory                |
| staticdir     | str  | *True*   | Full path to the static files directory       |
| templatedir   | str  | *True*   | Full path to the templates directory |
| timeout       | int  | *False*  | Seconds a request will wait before timing out  (default: 5) |

### Example Configurations

#### Without Cache
```json
{
    "hosts": {
        "somehost.example.com": "prod",
        "another.host.example.com": "prod",
        "aqasystem.example.com": "qa",
        "127.0.0.1": "dev"
    },

    "extranotes": "http://munin.mysite.com/%s.html",
    "endpoint": "http://%s:8008/",
    "templatedir": "/var/www/talook/templates/",
    "logdir": "/var/logs/talook/",
    "staticdir": "/srv/www/talook/static/",
    "timeout": 3
}
```

#### With Cache
```json
{
    "hosts": {
        "somehost.example.com": "prod",
        "another.host.example.com": "prod",
        "aqasystem.example.com": "qa",
        "127.0.0.1": "dev"
    },

    "extranotes": "http://munin.mysite.com/%s.html",
    "endpoint": "http://%s:8008/",
    "templatedir": "/var/www/talook/templates/",
    "cachedir": "/var/cache/talook/",
    "cachetime": {"hours": 1},
    "logdir": "/var/logs/talook/",
    "staticdir": "/srv/www/talook/static/",
    "timeout": 3
}
```

## URLS

### /
Index page. What a user will interact with.

### /hosts.json
Returns JSON data listing all configured hosts.

### /envs.json
Returns JSON data listing all configured environments.

### /host/*$HOSTNAME*.json
Returns stats for a specific host in JSON format. Cache is used if available.

### /statict/*$FILENAME*
Returns a static file from the static directory.

### /#!/host/*$HOSTNAME*
Returns stats for a specific host in the UI. This is helpful for bookmarking hosts.


## Logging
There are two log file which are produced by a running instance.

* **talook_access.log**: Access log similar to apache's access log.
* **talook_app.log**: Application level logging which logs some logic results.


## Running

### Simple
1. Edit the configuration file
2. Check to see what options you want to use with `--help`
3. `python server.py --listen 0.0.0.0 --port 8008 --config ./config.json`

#### Standalone Server Options
```
$ ./server.py --help
Usage: server.py [options]

Options:
  -h, --help            show this help message and exit
  -c CONFIG, --config=CONFIG
                        Config file to read (Default: config.json
  -p PORT, --port=PORT  Port to listen on. (Default: 8080)
  -l LISTEN, --listen=LISTEN
                        Address to listen on. (Default: 0.0.0.0)
  -r, --reload          Enable reloading on config change. (Default: False)
```


### In Apache
**mod_wsgi** can be used with Apache to mount talook. While the
standalone server will work just fine for some environments it's
important to remember it's single threaded and won't perform well
under some conditions. There are example files in `contrib/apache/`
which can help set an instance up. **Note** that the wsgi process
owner will need to be able to write and/or read from the locations
listed in the `config.json` just like in the standalone server!

* **talook.wsgi**: The WSGI file that mod_wsgi will use.
* **talook.conf**: The configuration file which mounts the WSGI application.

#### SELinux
One or both may be needed when using **mod_wsgi** on Apache if SELinux is enabled.

* `setsebool -P httpd_can_network_connect 1`
* `semanage port -a -t http_port_t -p tcp 8008`


### Packaging

#### RPM
To generate an RPM issue the following command:

```
$ make rpm
```

You'll then be able to install and run talook as a system service.
