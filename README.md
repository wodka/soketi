pWS - Pusher (over) [uWS](https://github.com/uNetworking/uWebSockets.js)
========================================================================

![CI](https://github.com/soketi/pws/workflows/CI/badge.svg?branch=master)
[![codecov](https://codecov.io/gh/soketi/pws/branch/master/graph/badge.svg)](https://codecov.io/gh/soketi/pws/branch/master)
[![Latest Stable Version](https://img.shields.io/github/v/release/soketi/pws)](https://www.npmjs.com/package/@soketi/pws)
[![Total Downloads](https://img.shields.io/npm/dt/@soketi/pws)](https://www.npmjs.com/package/@soketi/pws)
[![License](https://img.shields.io/npm/l/@soketi/pws)](https://www.npmjs.com/package/@soketi/pws)

pWS is an open-source Pusher protocol-ready, WebSockets-based server, that is container-ready and scalable to fit your needs to test Pusher locally.

The server is built on top of [uWebSockets.js](https://github.com/uNetworking/uWebSockets.js), a (ported to Node.js) C application that claims to be running _[8.5x that of Fastify](https://alexhultman.medium.com/serving-100k-requests-second-from-a-fanless-raspberry-pi-4-over-ethernet-fdd2c2e05a1e) and at least [10x that of Socket.IO](https://medium.com/swlh/100k-secure-websockets-with-raspberry-pi-4-1ba5d2127a23). ([source](https://github.com/uNetworking/uWebSockets.js))_

This project nor the owner is NOT affiliated with Pusher and it's strictly an open-source project that makes use of the [Pusher Protocol](https://pusher.com/docs/channels/library_auth_reference/pusher-websockets-protocol).

- [pWS - Pusher (over) uWS](#pws---pusher-over-uws)
  - [What pWS does not implemented (yet) from the Pusher Protocol?](#what-pws-does-not-implemented-yet-from-the-pusher-protocol)
    - [REST API](#rest-api)
      - [`/apps/[app_id]/channels/[channel_name]`](#appsapp_idchannelschannel_name)
      - [`/apps/[app_id]/channels`](#appsapp_idchannels)
  - [🤝 Supporting](#-supporting)
  - [System Requirements](#system-requirements)
  - [🚀 Installation](#-installation)
  - [🙌 Usage](#-usage)
  - [⚙ Configuration](#-configuration)
    - [📀 Environment Variables](#-environment-variables)
    - [📡 Pusher Compatibility](#-pusher-compatibility)
    - [🎨 Client Configuration](#-client-configuration)
    - [👓 App Management](#-app-management)
    - [↔ Horizontal Scaling](#-horizontal-scaling)
  - [📦 Deploying](#-deploying)
    - [🚢 Deploying with PM2](#-deploying-with-pm2)
    - [🐳 Deploying with Docker](#-deploying-with-docker)
    - [⚓ Deploy with Helm](#-deploy-with-helm)
    - [🌍 Running at scale](#-running-at-scale)
  - [🤝 Contributing](#-contributing)
  - [🔒  Security](#--security)
  - [🎉 Credits](#-credits)

## What pWS does not implemented (yet) from the Pusher Protocol?

The list may be incomplete. To address any problem that is **NOT known** (aka from the list below), please open an issue from the issue board.

### REST API

- HTTP Keep-Alive
- POST Batch Events endpoint
- POST Events and Batch Events does not return channel data in return
- Webhooks are not **yet** supported

#### `/apps/[app_id]/channels/[channel_name]`

- The endpoint does not support `info` query parameter

#### `/apps/[app_id]/channels`

- The enedpoint does not support `filter_by_prefix` and `info` query parameters
- `user_count` parameter for presence channels is not implemented

## 🤝 Supporting

Renoki Co. and Soketi on GitHub aims on bringing a lot of open source projects and helpful projects to the world. Developing and maintaining projects everyday is a harsh work and tho, we love it.

If you are using your application in your day-to-day job, on presentation demos, hobby projects or even school projects, spread some kind words about our work or sponsor our work. Kind words will touch our chakras and vibe, while the sponsorships will keep the open source projects alive.

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/R6R42U8CL)

## System Requirements

The following are required to function properly:

- Node 14.0+
- Redis 6+ (for running at scale)

## 🚀 Installation

You can install the package via npm:

```bash
npm install -g @soketi/pws
```

For additional ways of deployment, see [Deploying](#-deploying).

## 🙌 Usage

You can run the pWS Server directly from the CLI:

```bash
$ pws-server start

# in case you want to run with PM2
$ pws-pm2-server start
```

## ⚙ Configuration

### 📀 Environment Variables

You may declare the parameters using environment variables directly passed in the CLI or at the OS level, or as key-value attributes in an `.env` file at the root of the project:

```bash
$ DEBUG=1 pws-server start
```

When placing the variables in an `.env` file, you can prefix them with `PWS_` to avoid conflicts with other apps, but consider this optionally:

```bash
# Within your .env file
PWS_DEBUG=1
```

```bash
# you no longer need to specify the `DEBUG` variable
$ pws-server start
```

Read the extensive [environment variables documentation](docs/ENV.md) and learn how to configure the entire pWS Server application to suit your needs.

### 📡 Pusher Compatibility

The server is entirely compatible with the [Pusher Protocol v7](https://pusher.com/docs/channels/library_auth_reference/pusher-websockets-protocol#version-7-2017-11) and tries to keep up with the [HTTP REST API reference](https://pusher.com/docs/channels/library_auth_reference/rest-api/) as fast as possible.

### 🎨 Client Configuration

Pusher clients are fully compatible with the WebSocket protocol implemented in this project. You just have to point the client to the server address:

```js
const PusherJS = require('pusher-js');

let client = new PusherJS(key, {
    wsHost: '127.0.0.1',
    wsPort: 6001,
    // wssPort: 6001,
    forceTLS: false, // unless SSL is enabled
    encrypted: true,
    disableStats: true,
    enabledTransports: ['ws', 'wss'],
});

client.subscribe('chat-room').bind('message', (message) => {
    //
});
```

### 👓 App Management

The apps are used to allow access to clients, as well as to be able to securely publish events to your users by using signatures generated by an app secret.

Get started with the extensive [documentation on how to implement different app management drivers](docs/APP_MANAGERS.md) for your app, based on your needs.

### ↔ Horizontal Scaling

pWS is optimized to work in multi-node or multi-process environments, like Kubernetes or PM2, where horizontal scalability is one of the main core features that can be built upon. The main concern when scaling horizontally is **how can i make the nodes to communicate between them?**.

To be able to scale it horizontally and efficiently enable node-to-node or process-to-process communication, pWS leverages a Redis connection

You can configure pWS to run with Redis as a primary adapter for the local-persistent data with [an environment variable](docs/ENV.md#adapters):

```bash
$ ADAPTER_DRIVER=redis pws-server start
```

## 📦 Deploying

### 🚢 Deploying with PM2

PM2 is not necessarily horizontally scalable by node, but it is by process. pWS is PM2-ready and [can scale to a lot of processes](docs/PM2.md) just like any other application, as long as you have [configured the adapter to run in horizontal scaling](#-horizontal-scaling).

### 🐳 Deploying with Docker

Automatically after each release, a Docker tag is created with the application running in Node.js. You might want to use them to deploy or test pWS on Docker, Docker Swarm or Kubernetes, or any other container orchestration environment. [Get started with the versioning & Docker usage on DockerHub](https://hub.docker.com/r/soketi/pws).

### ⚓ Deploy with Helm

pWS can also be deployed as Helm v3 chart. pWS has an official Helm chart at [charts/pws](https://github.com/soketi/charts/tree/master/charts/pws). You will find complete installation steps to configure and run the app on any Kubernetes cluster.

### 🌍 Running at scale

If you run pWS standalone in a cluster, at scale, you might run into capacity issues: RAM usage might be near the limit and even if you decide to horizontally scale the pods, new connections might still come to pods that are near-limit and run into OOM at some point.

Running [Network Watcher 4.0+](https://github.com/soketi/network-watcher) inside the same pod will solve the issues by continuously checking the current pod using the Prometheus client (**Prometheus Server is not needed!**), labeling the pods that get over a specified threshold.

## 🤝 Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## 🔒  Security

If you discover any security related issues, please email alex@renoki.org instead of using the issue tracker.

## 🎉 Credits

- [Alex Renoki](https://github.com/rennokki)
- [Pusher Protocol](https://pusher.com/docs/channels/library_auth_reference/pusher-websockets-protocol)
- [All Contributors](../../contributors)
