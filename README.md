# babbys first containerz

Docker container based development for three-year-olds

## what is the point of this repo?

I was helping a friend set up his middleware-app + database on a $5/mo server. I was able to set the database up quickly 
enough, and connect his app to it, but got stuck when putting everything behind a lets-encrypt reverse proxy.

Furthermore, the entire process seemed pretty confusing from a beginners' perspective.
 
This project is an attempt at creating a containerization development environment even babbys' could use.

## I am literally a baby, how do I use this to develop a containerized application?

This infrastructure is split into two main parts; the main directory and `/app`. an example app is found in 
`/app/example/middleware`. an example database service used by the app is found in `/app/example/database`. I might do an example with them together as well, I haven't decided.

## foundational baby block: set-up node and domain name forwarding to node 

Get a node and a domain name and set that up properly

1. get a $5 digital ocean node
2. create an A record for `your-awesome-domain-name.com` pointing at your DO node IP
3. create a CNAME record for your domain with `*` to `your-awesome-domain-name.com` to allow all subdomains

> DNS changes can take a while, which can cause automatic ssl to not work properly. I will add a backdoor to make this easier to test later by adding another docker-compose file. This additional docker-compose feil will expose external backdoor ports to portainer and traefik.

## supportive baby block: setting up automatic ssl-termination and portainer for support

this sets up ssl termination so when you go to `your-awesome-domain-name.com` it doesnt give a security warning, and
also sets up `portainer` as a tool even babys can use to monitor and deploy docker containers

2. ssh into the DO node, or better yet, connect to the DO node using `Visual Studio Code SSH-Remote` with `[cmd + shift + p] type 'Remote-ssh:Connect to host'`
3. clone this repo `git clone https://github.com/WilliamTheMarsman/babys-first-containerz.git`
4. replace `codingwhileblack.xyz` with `your-awesome-domain-name.com` in `traefik.toml` and `docker-compose.yml`.
7. run `docker-compose up -d`
8. visit `portainer.your-awesome-domain-name.com` and **change the admin password right away**
9. visit `traefik.codingwhileblack.xyz` to see your reverse-proxy stuff

if you can't get to `portainer.your-awesome-domain-name.com`, check the logs for traefik with `docker logs traefik` on the DO node. if you can get to `portainer.your-awesome-domain-name.com`, this is the best place to check the logs.

## structural baby block: setting up a database type service or other private services your app needs

So there's two options here; if its a regular database, its easy because it can be deployed from 
`portainer.your-awesome-domain-name.com` and it doesn't need to be exposed to the outside world at all, which is great.

alternative, a docker-compose file could be used, but ultimately either is fine.

> Not sure what to put in your app instead of `localhost`? well, whatever name is given to the container ALSO 
corresponds to the 'in-container-cluster' network DNS name! So if you deploy a container with name `neo4j`, that
also becomes the dns name used to connect! ie from `bolt://localhost:7474` to `bolt://neo4j:7474`.

However, if its a database which has a UI component, that's probably not good enough, and some kind of endpoint might
need to be exposed.

This can be done using labels. It's pretty easy but I am pretty tired right now so I'm not going into it much.
Basically, you use docker labels to specify how `traefik` should expose the service to the outside world. Here's
the details in the [traefik docs for docker containers](https://docs.traefik.io/configuration/backends/docker/#on-containers)

Alternatively, this would probably work as docker labels for the UI component of the database:

- "traefik.enable=true"
- "traefik.port=8080" (specify port which has the UI or whatever)
- "traefik.frontend.rule=Host:database.your-awesome-domain-name.com"

## final baby block: building and deploying app middleware

With all the supporting services stood up, we can add the final baby-block to this project.

5. change your app to point to stuff like `neo4j` instead of `localhost`
1. build your app (can skip optional if building it on the node)
2. build a docker image for your app (can skip optional if building it on the node)
3. (opt) push docker image to a private docker registry in gitlab (or somewhere else)
4. (opt) add your private docker registry in portainer to allow pulling the docker image
5. use portainer or a docker-compose file to deploy your app docker image, with docker labels:
  - "traefik.enable=true"
  - "traefik.port=3000"
  - "traefik.frontend.rule=Host:your-awesome-domain-name.com,Host:www.your-awesome-domain-name.com" 
  - "traefik.frontend.priority=5" 

you're done. your app is now deployed, `https` is working and your database is running in the background. you can use portainer to add more services, monitor your application and other stuff.

## but what if baby wants to see app logs because something is not working?

app logs can be seen in `portainer.your-awesome-domain-name.com`, navigate to `containers>(container)>logs`

## but what if baby wants to update the app?

repeat the process for the final baby block. when you attempt to redeploy using portainer, it will replace the old image with the new one.

## but what if baby wants to deploy another middleware service

[read the traefik docs](https://docs.traefik.io/configuration/backends/docker/#on-containers) to make sure you know what you're doing and then do that.

## but what if baby wants deploy on commit

baby doesn't get deploy on commit

> let me know if there's an easy way to set this up. seems overkill for this kind of 'toy' set-up though

## but what if baby wants a cluster of vms?

baby doesn't get a cluster of vms. buy a bigger node.

## License

Apache 2.0
