---
date: '2025-12-21T14:42:09Z'
draft: false
title: 'Deploy HUGO static website using docker'
tags: ["docker"]
author: ["Paolo Sofia"]
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: ""
canonicalURL: ""
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

Many tutorials on the web focus on deploying hugo static websites on bare metal or using vps. In my case, I wanted to deploy this blog as a docker container using docker compose, as I do with all the docker services I deploy in my home server.
I faced some small problems while figuring out how to make it work, so I decided to document it here, so that in case someone is in the same situation as mine, can hopefully find this useful.


### Step 1. Create the docker compose
This is the easiest step, which is generating the docker compose file. Below is the `docker-compose.yml` file I am currently using, and in the next sections I will show you the tricky parts, which is the command section. For now, as you can see, there not a lot of settings to configure, I only had to add the traefik labels to expose the website to the internet, and mount the volume. 

```yml
services:
  hugo:
    image: ghcr.io/gohugoio/hugo:latest
    container_name: hugo
    command: server --bind 0.0.0.0 -b https://blog.paolosofia.ovh/ --appendPort=false --disableFastRender --cleanDestinationDir
    ports:
      - "1313:1313" 
    volumes:
      - "/path/to/webiste/name_of_the_website:/project" # mount your site here after you've created a new site!
    networks:
      - server
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.hugo-http.entrypoints=web"
      - "traefik.http.routers.hugo-http.rule=Host(`blog.paolosofia.ovh`)"
      - "traefik.http.routers.hugo-https.entrypoints=websecure"
      - "traefik.http.routers.hugo-https.rule=Host(`blog.paolosofia.ovh`)"
      - "traefik.http.routers.hugo-https.tls=true"
      - "traefik.http.routers.hugo-https.tls.certresolver=cloudflare"
      - "traefik.http.services.hugo-http.loadbalancer.server.port=1313"
      - "traefik.docker.network=server"      

networks:
  server:
    name: server
    external: true
```
### 2. Edit the command to execute
I encountered 3 problems when first deploying this docker compose, they are easy to fix but it took a bit to figure it out.

#### 2.1 Edit bind address
The first problem was with the bind address, since the docker-compose.yml I found on the internet was using 127.0.0.1 as bind address.  
In order to make hugo work in docker, you must pass the address 0.0.0.0, otherwise you will receive a 502 error.

#### 2.2 Set the base URL
If you read hugo's documentation, you most probably have set the baseURL in the hugo.toml or hugo.yml file. That is not sufficient for deploying the static website using docker. You need to pass the baseURL in the command section. If you don't do so, the base url will be set as localhost. 

When I deployed the first time after fixing the bind address, I was able to see the main page at the baseurl, but every link was pointing to localhost.

#### 2.3 Set appendPort to false
By default, the *appendPort* parameter is set to true, causing every link to explicitly include the port. In my case every link was pointing to https://blog.paolosofia.ovh:1313, which caused the browser to fail reaching the page, resulting in a timeout error.

### 3 Create the website before deploying
Before deploying this compose file, you must create the hugo website as you would when not using docker. To do so, you can run this command:

```sh
docker run --rm -v /path/to/webiste:/project ghcr.io/gohugoio/hugo:latest new site name_of_the_website --format yaml
```

After running the command, if successful, you will see a new folder with the website content, and that means that the website folder has been created. You can now deploy the compose file using `docker compose up -d`

### 4 Create new content
When creating new content, you can either manually create a md file in the *content* folder, or run the hugo **new content** command that created the file for you, applying the [archetype](https://gohugo.io/content-management/archetypes/). Run this command if you want to use this latter solution:

```sh
docker run --rm -v /path/to/webiste/name_of_the_website:/project ghcr.io/gohugoio/hugo:latest new content posts/post_name.md
```

### 5 Add cleanDestinationDir to remove deleted content
If you don't add this flag, then the *public* folder containing all the html code will never be cleaned, meaning that even if you remove a post from the *content* folder, it will still be deployed and still be visible on the website.

I noticed that after adding this flag, the live reloading feature started to work, which for whatever reason to me unknown, it was not working before. I don't know if this flag is direcly linked to the live reloading feature, so check it out if you see that the live reloading feature is not working.
