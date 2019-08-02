## Introduction

This docker stack will run many services (Traefik (with auth), Socat, Portainer, Nginx, Caddy, Whoami) in a straightforward copy-paste command. Please also refer the [README](https://github.com/pascalandy/docker-stack-this/blob/master/README.md) at the root of this repo.

## What news since `traefik_stack5`

wip

## Start here

1. Go to http://labs.play-with-docker.com/ 
2. Create *a cluster* using the the one click tool

![launch-a-cluster](https://user-images.githubusercontent.com/6694151/62176506-cd320700-b30f-11e9-818d-07bd43ea78ec.gif)

3. On one of the manager node, copy-paste:

#### stable branch (recommended)

```
ENV_BRANCH="master";
ENV_MONOREPO="traefik_stack6";

echo "Setup the stack" && \
source <(curl -s https://raw.githubusercontent.com/pascalandy/docker-stack-this/${ENV_BRANCH}/${ENV_MONOREPO}/play-with-docker-setup.sh) && \
sleep 2 && \
git checkout ${ENV_BRANCH} && \
cd ${ENV_MONOREPO} && \
./runup.sh;
```

#### edge branch (NOT recommended)

```
ENV_BRANCH="edge";
ENV_MONOREPO="traefik_stack6";

echo "Setup the stack" && \
source <(curl -s https://raw.githubusercontent.com/pascalandy/docker-stack-this/${ENV_BRANCH}/${ENV_MONOREPO}/play-with-docker-setup.sh) && \
sleep 2 && \
git checkout ${ENV_BRANCH} && \
cd ${ENV_MONOREPO} && \
./runup.sh;
```

This will run `play-with-docker-setup.sh` and `runup.sh`. These scripts will do the hard of deploying the stacks for us.

#### See your stacks

```
docker stack ls

NAME                SERVICES            ORCHESTRATOR
toolconsul          2                   Swarm
toolgui             2                   Swarm
toolproxy           2                   Swarm
toolwebapp          4                   Swarm
```


#### See your services

```
docker service ls

ID                  NAME                        MODE                REPLICAS            IMAGE                            PORTS
ruapy5e7ru6u        toolconsul_consul-leader    replicated          1/1                 consul:1.5
hvc4ssxz5rme        toolconsul_consul-replica   replicated          3/3                 consul:1.5
t4e2094q5v6o        toolgui_agent               global              5/5                 portainer/agent:latest
ywnm2dm6cx8s        toolgui_portainer           replicated          1/1                 portainer/portainer:latest
rtljih72vls4        toolproxy_socat             replicated          3/3                 devmtl/socatproxy:1.2
nu1d14ysu6c3        toolproxy_traefik           replicated          3/3                 traefik:1.7.12                   *:80->80/tcp, *: 443->443/tcp, *:8080->8080/tcp
cbov3qbzn3xo        toolwebapp_home             replicated          3/3                 abiosoft/caddy:0.11.1-no-stats
zfnnpl4iql80        toolwebapp_who1             replicated          3/3                 nginx:1.15-alpine
h5dn0jmk9he1        toolwebapp_who2             replicated          1/1                 emilevauge/whoami:latest
1nztlij8emje        toolwebapp_who3             replicated          1/1                 emilevauge/whoami:latest
```

## Confirm that your services (containers) are running

1. When you see that all services are deployed, click on `80` to see the static landing page.
2. From the same URL generated by play-with-docker, in the address bar of your browser, add `/who1/` or `/who2/` or `/who3/` or `/portainer/` to access other services.


#### Full URL example

```
http://ip172-18-0-3-bl0dq2l35dvg00f09hcg-80.direct.labs.play-with-docker.com/
http://ip172-18-0-3-bl0dq2l35dvg00f09hcg-80.direct.labs.play-with-docker.com/who1/
http://ip172-18-0-3-bl0dq2l35dvg00f09hcg-80.direct.labs.play-with-docker.com/who2/
http://ip172-18-0-3-bl0dq2l35dvg00f09hcg-80.direct.labs.play-with-docker.com/who3/
http://ip172-18-0-3-bl0dq2l35dvg00f09hcg-80.direct.labs.play-with-docker.com/portainer/
```

The container for the first URL is named `home`.


#### Web apps details:
- **/** = [caddy](https://github.com/pascalandy/caddy-securityheader)
- **/who1** = [caddy](https://github.com/pascalandy/caddy-securityheader)
- **/who2** = [whoami](https://hub.docker.com/r/emilevauge/whoami/)
- **/portainer/** = [portainer](https://hub.docker.com/r/portainer/portainer/)

For /who1 and /who2 you will see the container's Ids (5fe91baf7a3a & 78a0c7287df1) in this example

```
$ docker ps | grep whoami
4114379237a9        emilevauge/whoami:latest   "/whoamI"                8 minutes ago       Up 8 minutes        80/tcp
                                    toolwebapp_who3.1.qq6cpdk9mw5j3ngspzm53xhqd
```


## How to access Traefik

![traefik](https://user-images.githubusercontent.com/6694151/50121682-86334d80-0227-11e9-8f25-93dd8714d306.jpg)


#### Traefik password

**user**: admin / **pass**: changethispass

This password is encrypted in our configs `.configs/traefik.toml`

To quickly generate your password using htpasswd, use my container:

```
docker run --rm -it devmtl/alpinefire:3.8-D sh -c 'htpasswd -Bbn admin changethispass'  
``` 

This will display:

``` 
admin:$2y$05$pAfipn3.brdHMI2eWGnYH.84XYqLozp1sUPi36/l54UAwv.zGLtNC
```

Insert this string in the `stack-proxy.yml` and `stack-consul.yml`

#### What is Traefik?

[Traefik](https://docs.traefik.io/configuration/backends/docker/) is a powerful layer 7 reverse proxy. Once running, the proxy will give you access to many web apps. I think this is a substantial use case to understand how this reverse-proxy works.

#### Traefik version 

In `toolproxy.yml` look for something like `traefik:1.7.12-alpine`.

In some mono-repo, I **my own traefik image**. Feel free to use the official images. It will not break anything.

#### Other stuff to know?

- This stack does not use ACME (https://). ACME is a pain while developing … reaching limits, etc.
- If you don’t want to use socat, check out the monorepo `traefik-manager-noacme`

## Screenshots

![docker-stack-this-stack5_11](https://user-images.githubusercontent.com/6694151/34073735-76c60ae2-e26e-11e7-85a1-755a7177b3f2.jpg)
![docker-stack-this-stack5_12](https://user-images.githubusercontent.com/6694151/34073736-76d461c8-e26e-11e7-9aea-c8dbc049a383.jpg)
![docker-stack-this-stack5_13](https://user-images.githubusercontent.com/6694151/34073737-76e1d998-e26e-11e7-8b7c-c619e91adadd.jpg)
![docker-stack-this-stack5_14](https://user-images.githubusercontent.com/6694151/34073738-76f163ae-e26e-11e7-86d7-27ea62ae3284.jpg)
![docker-stack-this-stack5_15](https://user-images.githubusercontent.com/6694151/34073739-77006d4a-e26e-11e7-8f2e-cbd4268ea403.jpg)
![docker-stack-this-stack5_16](https://user-images.githubusercontent.com/6694151/49540846-158f4700-f89f-11e8-8e14-ceca2ff2b910.jpg)

![docker-stack-this-stack5_17](https://user-images.githubusercontent.com/6694151/49540848-1922ce00-f89f-11e8-9fdc-b6fce70825c8.jpg)

## All commands
In the active path, just execute those bash-scripts:

- `./runup.sh`
- `./rundown.sh`
- `./runctop.sh`

**Bonus!** `./runctop.sh` is not a stack but a simple docker run to see the memory consumed by each container.