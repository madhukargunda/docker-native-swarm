# docker-native-swarm

### This article i am going to discuss about the what is docker native swarm how its works

- How docker native swarm works
- Docker swarm example



## Why docker swarm

- We can run the docker containers in single node but when node/host fails all the applications which are running in node
  will also fail.No applications are going to work. With the single node we are not going to achieve the highavailability of     the applications.

- Hence docker introduced a concept called docker swarm, using this swarm we can create a group of nodes/docker-machines which 
  are running docker demaons grouped into one called as swarm cluster.
  
- Its responsibility of the docker swarm to distributing all the containers into different nodes to provide high 
  availability. 
  
- Docker swarm also provides the distributing the load in to different docker machines

- In docker swarm will have the Manager and Workers 

- Manager Nodes responsibilites are

        1.Maintaining the state of the containers
        2.Distributing the containers into different host machines
        3.Mainting the state of the cluster
        3.Managment of the cluster 
            - Adding the Workers Nodes
            - Removing the Worker Nodes
           
- In production environment singel node manager is not prefered

- In Production environment Multiple Manager nodes will be grouped in to one cluster and only 
  one manager node is responsibile making decesions .This node called as Leader
  
- Only Leadernode will take the all the decesions and informed to the other Manager Nodes

- When any of the situation Leader Node getting failed before updating the information to other Manager Nodes
  cluster will go inconsistent state. 
  
     Ex : 
     When Leader Nodes adding the Worker node or other Manager Node in middle of the update Leader fails other Manager will 
     not aware of the new changes and in future distributing newly added nodes will not be considered
  
- Docker uses the distributed raft consensus algorithoms to make cluster consistency.


### No docker-compose

- When there is no docker-compose we have to run the each and every command in each host node to create the container.
- Lets consider voting app and result app from docker sample application

  docker run -d --name=redis redis

  docker run -d --name=db postgres:9.4

  docker run -d -name=vote -p 5000:80 --link redis:redis voting-app

  docker run -d -name=result -p 5001:80 --link db:db result-app

  docker run -d --name=worker --link db:db --link redis:redis worker
  
- This little tricky to remember all these commands with docker compose this becomes very easy

- Using docker compose we can scale the containers (if the service is not bind to host port)

### docker services

![docker-service_replica](https://user-images.githubusercontent.com/5623861/56466944-38d02280-644b-11e9-91a3-f62b21506fe2.jpg)


### docker stack 

- To deploy the containers in production docker introduced the command as docker stack 
- stack consist of the services
- we can reuse the docker compose file for stack deployment
- deploy is the new key word introduced in docker stack we can specify 
    a.number of replicas we want
    b.replica where we want to place the in the docker swarm cluster (in which host node)
    c.restart policy of the container
    d.rolling updates for service
- when we use this docker compose on stack file , compose ignores the deploy command instructions
- docker stack doesnot supports the building the image. when we using the stack means docker thinks
  images is readily avaibale
- still we want build the image use the CI and CD plugins to build the images before using the stack
- compose ignores the deploy ,swarm ignores the build from compose yml file.
- We can use the same compose file for both the local and production deployments.


![docker-service-swarm](https://user-images.githubusercontent.com/5623861/56466965-91072480-644b-11e9-85b4-edc048ee3c4a.jpeg)


### Steps to create docker swarm 


```
Step1 : 

Create the docker-machine Manager node
   docker-machine create --driver virtualbox manager1

Step2 : 

Create the docker-machine Worker-1 and Worker-2 node
   docker-machine create --driver virtualbox worker1
   docker-machine create --driver virtualbox worker2

Step3 : 

connect to docker manager node
eval $(docker-machine env manager1)

Step4 : 

docker swarm init --advertise-addr $(docker-machine ip manager1)

Swarm initialized: current node (b4etd4utcgn9xnpawfs2tf5qe) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0hbyhpos06fwf85r35319wyhjpfysk9l88emio2av1qsr04alu-8th0xvea2f0yw3voiqc5er32c 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

Step5: 

Go to the other worker nodes and join the worker nodes to swaram manager
        eval $(docker-machine env worker-1)

docker swarm join --token SWMTKN-1-0hbyhpos06fwf85r35319wyhjpfysk9l88emio2av1qsr04alu-8th0xvea2f0yw3voiqc5er32c 192.168.99.100:2377

Step6 : 

eval $(docker-machine env worker-2)
docker swarm join --token SWMTKN-1-0hbyhpos06fwf85r35319wyhjpfysk9l88emio2av1qsr04alu-8th0xvea2f0yw3voiqc5er32c 192.168.99.100:2377

Step7: execute docker-machine ls

NAME            ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
load-balancer   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.09.5   
manager1        -        virtualbox   Running   tcp://192.168.99.100:2376           v18.09.5   
worker-1        -        virtualbox   Running   tcp://192.168.99.102:2376           v18.09.5   
worker-2        *        virtualbox   Running   tcp://192.168.99.103:2376           v18.09.5   

Step8 : 

Go to loadbalancer node
eval $(docker-machine env load-balancer)

Step9 :

download the docker-compose.yml file 

Step10 :

docker stack deploy -c docker-compose.yml voting-stack
Ignoring deprecated options:

container_name: Setting the container name is not supported.

Creating network voting-stack_back-end
Creating network voting-stack_front-end
Creating network voting-stack_default
Creating service voting-stack_voting-app
Creating service voting-stack_redis
Creating service voting-stack_worker
Creating service voting-stack_db
Creating service voting-stack_result-app
Creating service voting-stack_visualizer

$ docker stack ls
NAME                SERVICES            ORCHESTRATOR
voting-stack        6                   Swarm

docker stack services voting-stack
ID                  NAME                      MODE                REPLICAS            IMAGE                                          PORTS
3q08xxqgsvxa        voting-stack_worker       replicated          0/1                 dockersamples/examplevotingapp_worker:latest   
iwm2qrdp2k28        voting-stack_visualizer   replicated          1/1                 dockersamples/visualizer:stable                *:8080->8080/tcp
nvf3itv9mtl3        voting-stack_result-app   replicated          1/1                 dockersamples/examplevotingapp_result:before   *:5001->80/tcp
obkyo4ywhczi        voting-stack_voting-app   replicated          1/1                 dockersamples/examplevotingapp_vote:before     *:5000->80/tcp
pbqt2iz3dsvu        voting-stack_redis        replicated          1/1                 redis:alpine                                   
tiw3xqv8tuf0        voting-stack_db           replicated          0/1                 postgress:9.4  

```
<img width="1673" alt="docker-stack-voting-app" src="https://user-images.githubusercontent.com/5623861/56468757-3ed1fd80-6463-11e9-8472-de2f5636e81a.png">








