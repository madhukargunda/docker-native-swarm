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
  
- Docker uses the distributed raft consensus algorithoms to make cluster consistency
  







