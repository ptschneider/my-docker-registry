# my-docker-registry

## Background

Faster to have a network-local instance of docker registry to work against.

Just start it up ('docker-compose up -d'), tag an image and push it.

Push it real good.

## Docker Non-SSL Config

If you are running only on a local LAN, you can skip the SSL rigamarole with:

  /etc/docker/daemon.json: 
    { "insecure-registries":[ "docker01mxlinux.acme.local:5000" ] }

  then: systemctl restart docker

## Use Local Registry

To put images in:

 1) First pull (or build) an image in your local docker instance

    e.g. docker pull node:iron-alpine

 2) Tag it with the prefix info for your local registry

    e.g. docker tag node:iron-alpine docker01mxlinux.acme.local:5000/node:iron-alpine 

    n.b. you can change any part of the tag name at this point, but the prefix must refer to your new local registry

 3) Push the image with its new tag

    e.g. docker push docker01mxlinux.acme.local:5000/node:iron-alpine


### Local Registry Query

Use HTTP to get directory info on local registry contents:

curl -X GET http://docker01mxlinux.acme.local:5000/v2/_catalog

{"repositories":["cochise","node"]}

$ curl -X GET http://docker01mxlinux.acme.local:5000/v2/node/tags/list

returns: {"name":"node","tags":["iron-alpine"]}

curl -X GET http://docker01mxlinux.acme.local:5000/v2/cochise/tags/list

returns: {"name":"cochise","tags":["latest"]}

