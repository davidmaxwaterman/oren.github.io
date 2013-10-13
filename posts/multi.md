# Modular Website

![router2](http://25.media.tumblr.com/2e300fd39f1f99087292b8f58ee39062/tumblr_mia72sqn9w1r3gb3zo1_400.gif)

## TLDR:
Cutting YP.com into small pieces is really nice.

## What
We always try to modularize our code - break down a big function to smaller ones or chunk a feature to it's use-cases.
What about the application level? what if we could take YP.com and break it into multiple website?  
Here are 4 websites we might have: Home, SRP, MIP and Mybook. 

Before we continue I want to make sure we are talking about a website and not a web application. My definition of a website is a site that most of the rendering is done on the server.
I say most since some elements of the site might use AJAX and client-side rendering. If on the other hand your site is mainly a single-page-app, and you are doing most of the rendering and logic on the client, this idea of multiple websites doesn't apply to you.
Think of Amazon vs Gmail. The former is a website and the latter is an app. YP.com is closer to Amazon.

OK, back on topic. We are going to split YP into small websites, each one is autonomous - it has a git repo with dependencies, and it can be deployed by itself.
The dependencies for each site will be NPM packages and some of them will be shared with other sites (tracking, css, utilities).

## How

```bash
                        +---------+
                        |         | Load Balancing
                        |  Nginx  |
                        |         | SSL Termination
                        +---------+
                             +
                             |
                             v
+---------------------------------------------------------+
|                                                         |
|                       +---------+                       |
|                       |         |                       | router: Node.js app or Nginx
|                       | router  |                       | home/srp/mip/mybook: Node.js website
|                       |         |                       |
|               ------- +---------+ ------                |
|              /        /        |        \               |
|             /        /         |         \              |
|  +---------+   +---------+   +---------+  +---------+   |
|  |         |   |         |   |         |  |         |   |
|  |  home   |   |   srp   |   |   mip   |  | mybook  |   |
|  |         |   |         |   |         |  |         |   |
|  +---------+   +---------+   +---------+  +---------+   |
|   port 8001     port 8002     port 8003    port 8004    |
+---------------------------------------------------------+
                    Production host 1
```

The router can be a simple Node app or a reverse proxy such as Nginx. In this example our router will be a Node app, More specificaly we use the [http-proxy](https://github.com/nodejitsu/node-http-proxy) package written by Nodejitsu.
For the sake of simplicity (deployment and management) the router and the websites will live on the same host, each on it's own process, but it's possible to put each app on a different host.

So how does the router work? it pass each request to the right website based on the request url.
For example, we want to send '/', '/api', '/autosuggest' and '/nearby.html' to the home app, and '/mybook' to the mybook app, etc.

### Example of sub-apps
SRP project:

```bash
ypu-srp/
├── node_modules
│   ├── ypu-listing
│   │   ├── helpers.js
│   │   └── model.js
│   ├── ypu-tracking
│   ├── ypu-style
│   └── ypu-util
├── package.json
├── public
│   ├── css
│   └── js
└── server.js
```

Mybook project:

```bash
ypu-mybook/
├── node_modules
│   ├── ypu-mybook
│   │   ├── helpers.js
│   │   └── model.js
│   ├── ypu-tracking
│   ├── ypu-style
│   └── ypu-util
├── package.json
├── public
│   ├── css
│   └── js
└── server.js
```

Notice that tracking, style and util are dependencies of both projects. public/css and public/js contain project specific code.

### Deployment

We have mainly 2 options:

1. [Blue-green deployment](http://martinfowler.com/bliki/BlueGreenDeployment.html) - setup new hosts, deploy new code and point the load-balancer to those hosts.
1. Rolling deployment - Disconnect host-1 from load balancer, deploy app(s), reconnect host-1 and repeat with all hosts.

## Why

* Maintenance - easier to add feature/fix bug when working on smaller codebase.
* Flexible - each project might require different architecture/framework.
* Velocity - since each project can be deployed separately, new features or bug fixes hit production faster.
* Team structure - clear boundaries between features can lead to more focused teams.
* Scaling - sub-app that receives more traffic is scaled horizontally by adding more instances.
* Innovation - big codebase makes it hard to experiment with new tools/technologies.

## Try it
We wrote a [proof-of-concept](https://github.com/citrusbyte/ypu-router) of that approach.  
It uses 3 separate projects: router, SRP, and the current node project (Home, MIP and Mybook).