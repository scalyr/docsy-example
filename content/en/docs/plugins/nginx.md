---
title: "Nginx"
warning: "__Note:__ An *agent monitor plugin* is a component of the Scalyr Agent. To use a plugin,
           simply add it to the ``monitors`` section of the Scalyr Agent configuration file (``/etc/scalyr/agent.json``)."
beforetoc: ""
---

This agent monitor plugin records performance and usage data from an nginx server.

@class=bg-warning docInfoPanel: An *agent monitor plugin* is a component of the Scalyr Agent. To use a plugin,
simply add it to the ``monitors`` section of the Scalyr Agent configuration file (``/etc/scalyr/agent.json``).
For more information, see [Agent Plugins](/help/scalyr-agent#plugins).


## Configuring Nginx

To use this monitor, you will need to configure your nginx server to enable the status module. For details,
see the [nginx documentation](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html).

First, verify that your nginx server supports the status module. Execute the following command:

    nginx -V 2>&1 | grep -o with-http_stub_status_module

The output ``with-http_stub_status_module`` indicates that your server supports the status module. Otherwise,
you will need to either recompile nginx with the ``--with-http_stub_status_module`` flag, or upgrade to a full
version of nginx that has been compiled with that flag.

Next, you must enable the status module, by updating the ``server`` configuration section of your nginx server.
This section can either be found in the ``nginx.conf`` file, or the file in your ``sites-available`` directory
that corresponds to your site. For most Linux systems, these are located at ``/etc/nginx/nginx.conf`` and
``/etc/nginx/sites-available``.

Add the following configuration inside the ``server { ... }`` stanza:

    location /nginx_status {
      stub_status on;      # enable the status module
      allow 127.0.0.1;     # allow connections from localhost only
      deny all;            # deny every other connection
    }

This specifies that the status page should be served at ``http://<address>/nginx_status``, and can't be accessed
from other servers.

Each time the Scalyr agent fetches ``/nginx_status``, an entry will be added to the Nginx access log. If you wish to
prevent this, add the line ``access_log off;`` to the above configuration.

Once you make the configuration change, you will need to restart Nginx.  On most Linux systems, use the following
command:

    sudo service nginx restart

To verify that the status module is working properly, you can view it manually. Execute this command on the server
(substituting the appropriate port number as needed):

    curl http://localhost:80/nginx_status

If you have any difficulty enabling the status module, drop us a line at [support@scalyr.com](mailto:support@scalyr.com).


## Sample Configuration

Here is a typical configuration fragment:

    monitors: [
      {
          module: "scalyr_agent.builtin_monitors.nginx_monitor",
          status_url: "http://localhost:80/nginx_status"
      }
    ]

If your nginx server is running on a nonstandard port, replace ``80`` with the appropriate port number. For additional
options, see Configuration Reference.


## Configuration Reference

Option                   | Usage
``module``               | Always ``scalyr_agent.builtin_monitors.nginx_monitor ``
``id``                   | Optional. Included in each log message generated by this monitor, as a field named ``instance``. \
                                  Allows you to distinguish between values recorded by different monitors. This is especially \
                                  useful if you are running multiple nginx instances on a single server; you can monitor each \
                                  instance with a separate nginx_monitor record in the Scalyr Agent configuration.
``status_url``           | Specifies the URL -- in particular, the port number -- at which the nginx status module is served.
``source_address``       | Optional (defaults to '127.0.0.1'). The source IP address to use when fetching the server status.


accessLog:
## Uploading the nginx access log

If you have not already done so, you should also configure the Scalyr Agent to upload the access log
generated by nginx. Scalyr's nginx dashboard uses this log to generate many statistics.

For most Linux systems, the access log is saved in ``/var/log/nginx/access.log``. To upload, edit the
``logs`` section of ``/etc/scalyr-agent-2/agent.json``. Add the following entry:

    logs: [
       ...

    ***   {***
    ***     path: "/var/log/nginx/access.log",***
    ***     attributes: {parser: "accessLog", serverType: "nginx"}***
    ***   }***
    ]

Edit the ``path`` field as appropriate for your system setup.


## Viewing Data

After adding this plugin to the agent configuration file, wait one minute for data to begin recording. Then 
click the {{menuRef:Dashboards}} menu and select {{menuRef:Nginx}}. (The dashboard may not be listed until
the agent begins sending nginx data.) You will see an overview of nginx data across all servers where you are
running the nginx plugin. Use the {{menuRef:ServerHost}} dropdown to show data for a specific server.

See [Analyze Access Logs](/solutions/analyze-access-logs) for more information about working with web access logs.


## Log reference

Each event recorded by this plugin will have the following fields:

Field        | Meaning
``monitor``  | Always ``nginx_monitor``.
``instance`` | The ``id`` value from the monitor configuration.
``metric``   | The metric name.  See the metric tables for more information.
``value``    | The value of the metric.


## Metric reference

The table below describes the metrics recorded by the nginx monitor.

Metric                        | Description
``nginx.connections.active``  | The number of connections currently open on the server.
``nginx.connections.reading`` | The number of connections currently reading request data.
``nginx.connections.writing`` | The number of connections currently writing response data.
``nginx.connections.waiting`` | The number of connections currently idle / sending keepalives.