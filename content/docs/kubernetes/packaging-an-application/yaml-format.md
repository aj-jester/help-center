---
date: "2016-07-03T04:02:20Z"
title: "YAML Overview"
description: "An overview of the various sections of the Replicated YAML."
weight: "2601"
categories: [ "Packaging a Kubernetes Application" ]
index: ["docs/kubernetes", "docs"]
gradient: "kubernetes"
icon: "replicatedKubernetes"
aliases: [/docs/kubernetes/packaging-an-application/overview/]
---

Replicated will deploy an application that is defined in a YAML spec. We currently support deploying an application that uses the Replicated scheduler or deploying a Kubernetes application. Understanding how each of these will be installed and maintained is an important consideration when choosing the scheduler to use to deploy your application.

{{< linked_headline "Replicated API Version" >}}

At the top of the YAML file, regardless of the scheduler, there must be a Replicated API version. The current API version to use is {{< replicated_api_version_current >}}. Note: The [Changelog](https://release-notes.replicated.com/) tracks the API version.

```yaml
replicated_api_version: {{< replicated_api_version_current >}}
```

{{< linked_headline "App Basics" >}}

The next section includes some basic information about your application release including the app name.

```yaml
name: My Enterprise Application
```

{{< linked_headline "Detailed App Properties Description" >}}

The properties section includes definitions of some optional (but recommended) application properties. For a list of available properties see [Application Properties](/docs/kubernetes/packaging-an-application/application-properties). You will notice the `{{repl` escape sequence. This invokes a Replicated [template function](/docs/kubernetes/packaging-an-application/template-functions), which will be discussed in more detail soon.

```yaml
properties:
  app_url: http://{{repl ServiceAddress nginx }}
  console_title: My Enterprise Application
```

{{< linked_headline "Support Page" >}}

Replicated supports displaying custom markdown content on the Support page of the admin console. This can be defined in the console_support_markdown key.

```yaml
console_support_markdown: |
  Documentation for My Enterprise Application can be found [here](http://docs.my-enterprise-application.com).

  When contacting us for help, please download a Support Bundle (below) and attach it to the ticket.  The support
  bundle contains generic system information collected from this server.  It does _not_ contain any data from
  your instance of My Enterprise Application.
```

{{< linked_headline "Snapshots (Backups)" >}}

The snapshots key is available to to enable and configure [Snapshots](/docs/snapshots/overview). The following example will allow your customer to enable snapshots. You can customize which volumes should be backed up and specify a script to be run before the snapshot is collected.

```yaml
backup:
  enabled: true
  kubernetes:
    pvc_names: [ "redis-data-volume" ]
```

{{< linked_headline "CMD" >}}

The Replicated platform has some built in [commands](/docs/config-screen/commands/) that make writing your configuration much more powerful. In the cmds section you can write commands which we will use later.  These are useful to generate install-time values such as default certs/keys, randomized passwords, JWT token keys, etc.

```yaml
cmds:
- name: postgres_password
  cmd: random
  args:
  - "64"
```

{{< linked_headline "Custom Metrics" >}}

Replicated can display [custom metrics](/docs/kubernetes/packaging-an-application/custom-metrics/) sent from the running instance to the Admin Console by including the stats names in a custom_metrics key.

```yaml
custom_metrics:
- target: stats.gauges.myapp100.disk.*.*
  retention: "1s:10m,1m:20m,1h:30d"
  aggregation_method: "average"
  xfiles_factor: 0.3
- target: stats.gauges.myapp100.ping.*
  retention: "1s:7d"
  aggregation_method: "last"
  xfiles_factor: 0.6
```


{{< linked_headline "Customer Config Section" >}}

This section is where you can configure fields to gather input from the user. This input can be used to further configure your application. The values here can be used as inputs to container environment variables, config files, and more using [template functions](/docs/kubernetes/packaging-an-application/template-functions/) or tested for validity with [test procs](/docs/config-screen/test-procs/). The [config section](/docs/config-screen/config-yaml/) is comprised of configuration groups and items. These items will render as a form in the Settings screen of the Replicated admin console.

```yaml
config:
- name: hostname
  title: Hostname
  description: Ensure this domain name is routable on your network.
  items:
  - name: hostname
    title: Hostname
    type: text
    value_cmd:
      name: host_ip
      value_at: 0
    ...
```

{{< linked_headline "Admin Commands" >}}

Optionally you can expose [admin commands](/docs/kubernetes/packaging-an-application/admin-commands/) in your containers. To configure the commands, add the following section. This example will allow the customer to run the `redis-cli` command with any arbitrary arguments. The command will be executed only in the Docker containers that match image name and version as well as defined in the named component. A command that will work with this configuration is `replicated admin redis-cli info`. Replicated will find the appropriate node to run this command on; the customer can run these on the main admin console.

```yaml
admin_commands:
- alias: redis-cli
  command: [redis-cli]
  run_type: exec
  component: DB
  container: redis
```

{{< linked_headline "Custom Preflight Checks" >}}

A [preflight check](/docs/kubernetes/packaging-an-application/preflight-checks/) is a test that is run before installing and running an application. The test will analyze the system to determine if the environment meets the minimum requirements and provide feedback during installation if these requirements are not met.

```yaml
kubernetes:
  requirements:
    server_version: ">=1.9.3"
    api_versions: ["apps/v1", "core/v1"]
    cluster_size: "1"
    total_cores: "2"
    total_memory: 11.25GB
```
