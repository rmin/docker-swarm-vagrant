# docker-swarm-vagrant
A Vagrantfile to spin up [Docker Swarm](https://docs.docker.com/engine/swarm/) cluster with any number of nodes on [Ubuntu 18.04 LTS](http://releases.ubuntu.com/18.04/) boxes.

## Usage
Clone the repository on your workstation with [Vagrant](https://www.vagrantup.com/) installed:
```
git clone https://github.com/rmin/docker-swarm-vagrant.git
cd docker-swarm-vagrant
```

And start the 3 node multi-manager Docker Swarm deployment by executing:
```
vagrant up
```

If you want only the first node to be the manager export `SWARM_WORKER` ENV var before running vagrant.
```
export SWARM_WORKER=true
vagrant up
```

Now you can ssh into any node:
```
vagrant ssh swarm1

...
vagrant@swarm1:~$ sudo docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
ots6d1q8ft9dctsc8e6uvnnsz *   swarm1              Ready               Active              Leader              18.05.0-ce
b0wplasmn4wxee0kwmsvld4h3     swarm2              Ready               Active              Reachable           18.05.0-ce
ik4jtzqkug6akzj4u7m7lsuqi     swarm3              Ready               Active              Reachable           18.05.0-ce
```

### Options
By default it installs 3 boxes with 512MB memory and 1CPU each, and adds them to 192.168.80.X private network.
To change any of the options, export ENV vars in the command line before executing vagrant.
```
export SWARM_NODES=2
export SWARM_MEMORY=1024
export SWARM_CPU=2
export SWARM_IPPREFIX="192.168.95."
vagrant up
```

### Scaling
To add new nodes to the cluster in the future just increase the value of `SWARM_NODES` ENV var, and run `vagrant up` again. If you haven't used the default values before, make sure to provide other options as well.
```
export SWARM_WORKER=true
export SWARM_NODES=4
export SWARM_MEMORY=1024
export SWARM_CPU=2
export SWARM_IPPREFIX="192.168.95."
vagrant up
```
Don't export `SWARM_WORKER` if you want the new nodes to be manager nodes.

## License
MIT License

Copyright (c) 2018 @rmin
