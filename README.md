Docker Swarm
============

```shell
vagrant up
vagrant hostmanager
```

# Getting started

```shell
# Create a swarm
eth1_ip=$(ip address show eth1 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
docker swarm init --listen-addr "$eth1_ip:2377" --advertise-addr "$eth1_ip:2377"

vagrant ssh worker1 -- sudo docker swarm join --token SWMTKN-1-3y2h4y0skb8zytknr9n93ngiv8wadodhol3onuaogyqd0wz7hz-aw38pzv0yfbonk622irkjh8sz 192.168.56.8:2377
vagrant ssh worker2 -- sudo docker swarm join --token SWMTKN-1-3y2h4y0skb8zytknr9n93ngiv8wadodhol3onuaogyqd0wz7hz-aw38pzv0yfbonk622irkjh8sz 192.168.56.8:2377

# Join a worker node to the swarm
docker swarm join-token manager

vagrant ssh manager2 -- sudo docker swarm join --token SWMTKN-1-3y2h4y0skb8zytknr9n93ngiv8wadodhol3onuaogyqd0wz7hz-57xvjyfkgtwi2qk5mn2henop3 192.168.56.8:2377

# List the nodes in the swarm
vagrant ssh manager1 -- sudo docker node ls


vagrant ssh manager1 -- curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
vagrant ssh manager1 -- sudo docker stack deploy --compose-file=portainer-agent-stack.yml portainer

# List the services in the swarm
vagrant ssh manager1 -- sudo docker node ps $(docker node ls -q)

# Create a service
vagrant ssh manager1 -- sudo docker service create --name my_nginx --replicas 8 --publish published=8080,target=80 nginx


http://10.10.170.101:8080/
http://10.10.170.101:9000/#!/1/docker/swarm/visualizer
```

# other resource

https://github.com/tdi/vagrant-docker-swarm

```shell
docker service create --name web --replicas 1 --publish 8080:8080 darek/goweb:1.0
docker service  ps web
docker service scale web=4


# Now, let us verify it works, docker swarm will load balance requests to all of the service instances:

vagrant ssh worker1
sudo apt-get install httpie -y
http localhost:8080


# Let's do a rolling update of the service, first let's scale it to 10 - you already know how. First we will update the defintion of the service to say that updates will have a delay of 5s.

docker service update --update-delay 5s web

# Let's us update the service now:

docker service update --image darek/goweb:2.0 web

# Now docker will update service one by one with a 5s delay between the updates. If you want to introduce parallelism to updates you can specify if with the --update-parallelism flag.

docker service update --update-parallelism 2 web


docker service ps web
docker service inspect --pretty web

# After the update, you can verify that the version is really 2.0 with the httpie command. Let's now delete the service.

docker service rm web

```# docker-swarm
