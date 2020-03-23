pulp-webserver
==============

Install, configure, start, and enable a web server.

Currently, Nginx and Apache are supported. They are configured as a reverse proxy to the pulpcore-api
and pulpcore-content Gunicorn processes.


Variables:
----------

* `pulp_webserver_server` Set the webserver Pulp should use to reverse proxy with. Defaults to
  'nginx'.
* `pulp_content_port` Set the port the reverse proxy should connect to for the Content app. Defaults
  to '24816'.
* `pulp_content_host` Set the host the reverse proxy should connect to for the Content app. Defaults
  to '127.0.0.1'.
* `pulp_api_port` Set the port the reverse proxy should connect to for the API server. Defaults to
  '24817'.
* `pulp_api_host` Set the host the reverse proxy should connect to for the API server. Defaults to
  '127.0.0.1'.
* `pulp_configure_firewall` Install and configure a firewall. Valid values are 'auto', 'firewalld',
  and 'none'. Defaults to 'auto' (which is the same as 'firewalld', but may change in the future).

Plugin Webserver Configs:
-------------------------

The installer symlinks config fragments from plugin Python packages to either nginx or apache during
installation. These fragments typically provide additional url routing to either the Pulp API or
Pulp Content App. pulp_ansible has an example of such configs [here](https://github.com/pulp/
pulp_ansible/tree/master/pulp_ansible/app/webserver_snippets).

The Nginx config provides definitions for the location of the Pulp Content App and the Pulp API as
pulp-api and pulp-content respectively. To route the url `/pulp_ansible/galaxy/` to the Pulp API you
could use this definition in a snippet like:

```
location /pulp_ansible/galaxy/ {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;
    # we don't want nginx trying to do something clever with
    # redirects, we set the Host: header above already.
    proxy_redirect off;
    proxy_pass http://pulp-api;
}
```

The Apache config provides variables containing the location of the Pulp Content App and the Pulp
API as pulp-api and pulp-content respectively. Below is an equivalent snippet to the one above, only
for Apache:

```
ProxyPass /pulp_ansible/galaxy http://${pulp-api}/pulp_ansible/galaxy
ProxyPassReverse /pulp_ansible/galaxy http://${pulp-api}/pulp_ansible/galaxy
```

SSL Configuration
-----------------

By default self-signed certificates are generated and both Nginx and Apache will serve TLS on port
443.

Shared variables:
-----------------

* `ansible_python_interpreter`: **Required**. Path to the Python interpreter.

This role is **not tightly coupled** to the `pulp` role, but uses some of the same
variables. When used in the same play, the values are inherited from the `pulp`
role.

* `pulp_install_dir`: Location of a virtual environment for Pulp and its Python
  dependencies. **Required** if used in a separate play from the `pulp` role. Value
  must match the value used in the `pulp` role.
