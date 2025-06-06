---
layout: default
title: Using Meshery CLI
abstract: How to use mesheryctl
permalink: guides/mesheryctl/working-with-mesheryctl
language: en
type: guides
category: mesheryctl
abstract: Guides for common tasks while using Meshery's CLI, mesheryctl.
---

Meshery's command line interface is `mesheryctl`. Use `mesheryctl` to both manage the lifecyle of Meshery itself and to access and invoke any of Meshery's application and cloud native management functions. `mesheryctl` commands can be categorized as follows:

- `mesheryctl` - Global overrides and flags
- `mesheryctl app` - Cloud Native Application Management
- `mesheryctl filter` - Cloud Native Filter Management
- `mesheryctl mesh` - Cloud Native Lifecycle & Configuration Management
- `mesheryctl perf` - Cloud Native Performance Management
- `mesheryctl design` - Cloud Native Pattern Configuration & Management
- `mesheryctl system` - Meshery Lifecycle and Troubleshooting

## Configuring Meshery Deployments with meshconfig

_Meshconfig_ refers to a configuration file found at `~/.meshery/config.yaml`. Your meshconfig file must contain one or more `contexts` in order for any `mesheryctl system` command to work. Each context represents a Meshery deployment.

Each of the `system` commands are used to control Meshery's lifecycle like `system start`, `stop`, `status`, `reset` and so on.

## Meshery CLI FAQ

#### Question: What is the meshconfig?

Like kubeconfig for `kubectl`, meshconfig is the name of your `mesheryctl` config file. You can find your meshconfig file in its default location of `~/.meshery/config.yaml`. By default, `mesheryctl` will look to this location for your meshconfig. You can override the default location at anytime with the use of the global parameter, `--config <my-other-meshconfig>`.

#### Question: What is a context?

A meshconfig `context` represents a single Meshery deployment. Using `context`s, you can configure different Meshery deployments with environment-specific settings and you can easily switching between your individual Meshery deployments by updating your current-context.

#### Question: Why are contexts necessary?

Many Meshery users have more than one Meshery deployment. Contexts allow you to deploy different versions of Meshery, update your release channel subscription settings, selectively install one or more Meshery Adapters, and so on. Contexts allow you to configure your individual Meshery deployments.

#### Question: What is `current-context`?

`current-context` identifies the Meshery deployment that when any `mesheryctl` command is invoked will use the environment described in the `current-context`. You can switch between contexts. Only one context can be the `current-context`.

#### Question: What's the difference between contexts and environments?

Contexts configure Meshery deployments (server, adapters, operator and so on), while environments define a collection of Kubernetes clusters and cloud native infrastructure under management in Meshery.

#### Question: What does the default meshconfig look like?

The following template is used to create a config file from scratch. Not all of the following variables are required to be included. Some of the variables may have a null value or may be excluded (e.g. “adapters”).

#### Question: What is the importance of --config flag?

The `--config` flag is a global option that applies to all `mesheryctl` commands. It allows you to specify the location of a custom meshconfig file, overriding the default configuration. This config file is used to set up the `mesheryctl` context, which defines the configuration for a particular Meshery deployment.

```
contexts:
    <context1-name>:
      endpoint: <url to meshery server rest api>
      token: <name of token variable in this config file>
      platform: <type of platform: ”docker” or “kubernetes”>
      # Future: specify type of kubernetes (e.g. eks)
      adapters: <collection of names of Meshery adapters:
          “istio”,“linkerd”,”consul”,”nginx-sm”,”octarine”,”tanzu-sm”,”citrix-sm”,”kuma”,”osm”,”nsm”>

   <context2-name>:
    endpoint: <url to meshery server rest api>
    token: <name of token variable in this config file>
    platform: <type of platform: ”docker” or “kubernetes”>
    current-context: <context name>

tokens:
- name: <token1-name>
  location: <token-location>
- name: <token2-name>
  value: <token-value>
  # Future: allow embedding of token certificate
```

Try it out and see for yourself. Run `mesheryctl system context create test` and `mesheryctl system context view test`.

#### Question: How do endpoints work in meshconfig?

Endpoints specify the access URL for the Meshery UI, for a deployment. Endpoints are developed based on platform:

- Docker: Docker users can specify the endpoint in the meshconfig. The port specified in this will be used to generate the endpoint. The endpoint is of the form `http://localhost:port`, where `port` is taken from the meshconfig.
- Kubernetes: Deployments with kubernetes as the platform have an endpoint generated by service discovery using the Kubernetes API. This endpoint overwrites the endpoint specified in the meshconfig.

#### Question: Can I get an API token using mesheryctl?

Yes, if you need to establish a session with your Meshery Server, you can [authenticate using mesheryctl](/guides/mesheryctl/authenticate-with-meshery-via-cli), using `mesheryctl system login`.

## Advanced Installation

Users can control the specific container image and tag (version) of Meshery that they would like to run by editing their local _~/.meshery/meshery.yaml_ (a docker compose file).
Aligned with the Meshery container image, instead of leaving the implicit :stable-latest tag behind image: meshery/meshery, users will instead identify a specific image tag like so:

{% capture code_content %}bash
version: '3'
services:
  meshery:
    image: meshery/meshery:v0.5.0
    labels:
      - "com.centurylinklabs.watchtower.enable=true"{% endcapture %}
{% include code.html code=code_content %}

### Suggested Reading

For an exhaustive list of `mesheryctl` commands and syntax:

- See [`mesheryctl` Command Reference]({{ site.baseurl }}/reference/mesheryctl).

Guides to using Meshery's various features and components.

{% capture tag %}

<li><a href="{{ site.baseurl }}/installation/upgrades#upgrading-meshery-cli">Upgrading mesheryctl</a></li>

{% endcapture %}

{% include related-discussions.html tag="mesheryctl" %}

<!-- ## Related Guides

<div>
  <a href="{{ site.baseurl }}/guides/mesheryctl/configuring-autocompletion-for-mesheryctl">
    <div class="overview">Configuring Autocompletion for `mesheryctl`</div>
  </a>
  <p>Configure automatic completion of `mesheryctl` commands in your environment.</p>
</div>

<div class="wrapper" style="text-align: left;">
  <div>
  <a href="{{ site.baseurl }}/reference/mesheryctl">
    <div class="overview">Command Reference</div>
  </a>
  <p>Find an exhaustive list of commands and their syntax.</p>
</div>

<div>
  <a href="{{ site.baseurl }}/installation/upgrades">
    <div class="overview">Upgrade Guide</div>
  </a>
  <p>To upgrade <code>mesheryctl</code>, refer to the Upgrade Guide.</p>
</div>


</div> -->

<!--
## Installing `mesheryctl`

### Mac or Linux

Use your choice of homebrew or bash to install `mesheryctl`. You only need to use one.
### Homebrew

Install `mesheryctl` and run Meshery on Mac with Homebrew.

#### Installing with Homebrew

To install `mesheryctl`, execute the following commands:

 <pre class="codeblock-pre"><div class="codeblock">
 <div class="clipboardjs">
 brew install mesheryctl
 mesheryctl system start
 </div></div>
 </pre>

**Upgrading with Homebrew**

To upgrade `mesheryctl`, execute the following command:

 <pre class="codeblock-pre"><div class="codeblock">
 <div class="clipboardjs">
 brew upgrade mesheryctl
 </div></div>
 </pre>

#### Bash

**Installing with Bash**

Install `mesheryctl` and run Meshery on Mac or Linux with this script:

 <pre class="codeblock-pre"><div class="codeblock">
 <div class="clipboardjs">
 curl -L https://meshery.io/install | bash -
 </div></div>
 </pre>

**Upgrading with Bash**

Upgrade `mesheryctl` and run Meshery on Mac or Linux with this script:

 <pre class="codeblock-pre"><div class="codeblock">
 <div class="clipboardjs">
 curl -L https://meshery.io/install | bash -
 </div></div>
 </pre>

## Windows

### Installing the `mesheryctl` binary

Download and unzip `mesheryctl` from the [Meshery releases](https://github.com/meshery/meshery/releases/) page. Add `mesheryctl` to your PATH for ease of use. Then, execute:

 <pre class="codeblock-pre"><div class="codeblock">
 <div class="clipboardjs">
 ./mesheryctl system start
 </div></div>
 </pre>

### Scoop

Use [Scoop](https://scoop.sh) to install Meshery on your Windows machine.

**Installing with Scoop**

Add the Meshery Scoop Bucket and install:

 <pre class="codeblock-pre"><div class="codeblock">
 <div class="clipboardjs">
 scoop bucket add mesheryctl https://github.com/meshery/scoop-bucket.git
 scoop install mesheryctl
 </div></div>
 </pre>

**Upgrading with Scoop**

To upgrade `mesheryctl`, execute the following command:

 <pre class="codeblock-pre"><div class="codeblock">
 <div class="clipboardjs">
 scoop update mesheryctl
 </div></div>
 </pre>

-->

{% include discuss.html %}

