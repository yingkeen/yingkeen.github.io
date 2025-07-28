---
layout: post
title: "Using External NPM Modules in n8n Function Nodes (e.g., youtube-transcript)"
---

## TL;DR

To use external NPM modules like `youtube-transcript` in n8n Function nodes:

- Build a **custom Docker image** that installs the modules.
- Set `NODE_FUNCTION_ALLOW_EXTERNAL=*` [^1] to allow using them in code.
- Deploy and run your container using the custom image.

See the full guide below.

---

While working on an automation workflow using n8n, I wanted to use the `youtube-transcript` npm module inside a Function node. I wrote a simple script using `require('youtube-transcript')`, and it worked — but only after I manually installed the module inside the Docker container.

That was fine until I redeployed the container — and everything I installed disappeared. That’s when I realized I needed a persistent way to make additional npm modules available inside n8n.

So, here's what worked for me.

## The Problem

n8n let you use JavaScript in Function nodes, and even supports `require()` for importing packages. But:

- The default **n8n Docker image** only includes pre-installed modules.
- You **can’t use external npm packages** unless they’re already part of the image.
- Any `npm install` you do inside a running container is **lost on redeploy**.

## The Solution: Build a Custom Docker Image With Your NPM Modules

To permanently include extra npm modules, the best approach is to **build your own Docker image** based on the official n8n image, and install the required packages during the image build.

There are 3 main steps:

1. **Create a Dockerfile** to install your modules.
2. **Build and (optionally) push the image** to your Docker registry.
3. **Deploy and run** your container with the correct environment variable.

### Step 1: Create a `Dockerfile` with Your Required Packages

Here’s a simple example to install `youtube-transcript`, and even something like `ffmpeg` if needed:

```Dockerfile
FROM n8nio/n8n

USER root

RUN npm install -g youtube-transcript
RUN apk update && apk add --no-cache ffmpeg

USER node
```

Save this file as `Dockerfile`.


### Step 2: Build and Push (If Needed)

```bash
docker build -t your-custom-n8n .
```

### Step 3: Deploy with NODE_FUNCTION_ALLOW_EXTERNAL=* [^1]

This environment variable is required to allow the use of external modules in Function nodes:

```env
NODE_FUNCTION_ALLOW_EXTERNAL=*
```

_Refer to n8n documentation [here](https://docs.n8n.io/hosting/configuration/environment-variables/nodes/) and [here](https://docs.n8n.io/hosting/configuration/configuration-examples/modules-in-code-node/)_.

#### Example: Run locally with Docker

```bash
docker run -p 5678:5678 -e NODE_FUNCTION_ALLOW_EXTERNAL=* your-custom-n8n
```

#### Example: `docker-compose.yml`

```yaml
services:
  n8n:
    image: your-custom-n8n
    ports:
      - 5678:5678
    environment:
      - NODE_FUNCTION_ALLOW_EXTERNAL=*

# Add any other required configuration
```

---

That’s it — now we can use external npm packages like `youtube-transcript` directly in the custom code without manually installing them after each redeploy.

---
{: data-content="footnotes"}

[^1]: Using NODE_FUNCTION_ALLOW_EXTERNAL=* allows access to all globally installed NPM modules from within Function nodes. This can expose your workflows to unintended behavior if untrusted users have access to the workflow editor. For stricter control, you can specify only the modules you need, e.g., NODE_FUNCTION_ALLOW_EXTERNAL=youtube-transcript.