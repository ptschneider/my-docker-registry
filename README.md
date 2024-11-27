# docker-registry-cache

## Background

It is faster to have a network-local instance of docker registry to work against.

Further, if you are using a free account you have to limit your pulls.

Cache your images in your own and manage what you pull/push with hub.docker.com

## Setup

You will want to setup a separate disk to store registry data; it can quickly consume many gigs of data and you will want to manage that separately.

If a VM, create a disk; it may appear as /dev/sde. create a partition with fdisk and create a filesystem mkfs.ext3 /dev/sde1, mount it to /var/lib/docker-registry-cache or whatever name you like.


## Docker Non-SSL Config

If you are running only on a local LAN, you can skip the SSL rigamarole with:

  /etc/docker/daemon.json: 
    { "insecure-registries":[ "docker01mxlinux.acme.local:5000" ] }

  then: systemctl restart docker

also add:
    { "allow-nondistributable-artifacts": ["myregistrydomain.com:5000"] }

    if you might encounter DRM-encumbered images from vendors in your world (an uncommon problem); such layers will not be replicated by-default (silently)

## Use Local Registry

Deleting images is a pain; don't push more than you feel like deleting...

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

### Delete an image

Multiple steps:

#### Find and delete catalog entry by digest

Run the command `curl -sS <domain-on-ip>:5000/v2/_catalog`

For the item you want to delete, list tags: `curl -sS <domain-on-ip>:5000/v2/<repo>/tags/list`

For the target tag, retrieve the message digest: `curl -sS -H 'Accept: application/vnd.docker.distribution.manifest.v2+json' \
-o /dev/null \
-w '%header{Docker-Content-Digest}' \
<domain-or-ip>:5000/v2/<repo>/manifests/<tag>`

You will see an output `sha256:XXX`, where XXX is a big long alphanumeric.

Now, execute the command to delete the catalog entry: (replace <digest> with the whole 'sha:256XXX' string) 
`curl -sS -X DELETE <domain-or-ip>:5000/v2/<repo>/manifests/<digest>`

It will complain on error; silence is success.

#### run garbage collection

This is a generalized facility that is run on-demand. 

Run `docker exec -it container_id /bin/sh' to connect to your running registry.

Edit the file /etc/docker/registry/config.yml; verify the service name & port are correct, and add the lines for delete:enabled:true under the storage section

Run the command `registry garbage-collect -m /etc/docker/registry/config.yml` to actually run GC and delete the blobs. You should see a lot of status messages but no errors.

If you don't see a bunch of blobs deleted, something probably went wrong.
