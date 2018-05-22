# CI/CD Automation Process 
(Under Development)Includes Scripts for CI/CD using Jenkins, Docker and GitHub in AWS(Amazon Web Services) EC2 Cloud Infrastructure.

My following project demonstrate:

 * Project 1  Binding Docker Container(Environment and Code together) which can run indipendently in any system.
 * Project 2  Running the containers in cluster(Docker swarm & Kubernetes Cluster) for Highavailability, Horizontal Scalling & Rolling Updates.
 * Project 3  Building a CI/CD environment with (Jenkins & Git) push trigger where developers can deploy their code in production with a push.
 * Project 4  System configration and Deployment using Ansible

## Project 1  Manual steps for running WorkHub in Docker Containers 
First I am running the my WorkHub Webapp manually in docker containers, Push it in my DockerHub Repo and after that i will run it in AWS EC2 instance pulling from dockerhub, which besically shows the scenarios of shiping whole environment with code.

The workhub web-app from my repo is converted into docker image and it is uploaded in DockerHub.
The link to DockerHub repository is:
        
        https://hub.docker.com/r/manilpuri9/workhub/ 

1. Run pull command in your server: 
  
        docker pull manilpuri9/workhub      

2. Run the downloaded image as:
   
        docker run -it -p 80:5000 workhub:v1
   
  I. Go inside the container
   
    docker exec -it <containerID> bash
   
 II. Run the app.py file inside Workhub folder
   
    python app.py

Upto here the server is running without the database. Just to check if our server is serving properly.
Go to your browser and type localhost. You must be able to see the homepage of WorkHub.

And here is the image of database:
   
    https://hub.docker.com/r/manilpuri9/database/
   
3. Run pull command in your server:
   
        docker pull manilpuri9/database

If you pull mysql:v2.1 all the configration are done previously so you can skip to step 7
   
    docker pull manilpuri9/mysql:v2.1  


4. Run the downloaded image as:
   
        docker run --name sql -e MYSQL_ROOT_PASSWORD=my-secret-pw -v <path of your mysqldumpfile.sql>:/root/ -it mysql
   
 In my case:
   
        docker run --name sql -e MYSQL_ROOT_PASSWORD=my-secret-pw -v /home/manil/Documents/git/WorkHub/:/root/ -it mysql
   
5. Go inside the container
   
        docker exec -it <containerID> bash
   
6. Create a database using mysql as:
   
        mysql -u root -p myflaskapp < workhubdatadump.sql

7. Stop and remove the running docker container that we created on step 1
   
         docker stop <containerID>
         docker rm <containerID>

8. Start it again but this time we link it to the running database container that we ran in step 5
   
        docker run --link sql:mysql -it -p 80:5000 -v /home/manil/Documents/git/WorkHub:/root/ manilpuri9/workhub:v1

##### At this point you must have a running webapp Workhub in your machine at port 80. Go to the browser and type localhost

## 2. Running my portfolio site in Docker Swarm & Kubernetes Cluster
###   a) Docker Swarm
   Now for :
   * High Availability,
   * Horizontal Scaling,
   * Decentralized design, 
   * Load balancing,
   * Rolling updates
   We use docker swarm / container cluster.
   
   1. For this I am building a docker swarm with my portfolio site image. 
      I am running 2 EC2 Instances in AWS, Installing Docker in both of them, and pulling portfolio image in both of them (say Instance 1 & Instance 2)
      
            docker pull manilpuri9/portfolio     (on both instances)
      
   2. Making one instance as master and another as slave. Running following command in instance 1(master).
      
            hostname -I (getting the private IP of master)
            docker swarm init --advertise-addr 172.31.86.17
   
   This command generates a token as:
   
    docker swarm join --token SWMTKN-1-2599d37taxeucuxqq0tbraz11h2e29h8upc1znl7ash4r8wzvg-0okv1lyptyeqcknmejrvthe0j              172.31.86.17:2377
   
   3. Before running the docker swarm join command in slave, I opened the port 2377 on my Instance 1(master).and ran the           command on the Instance 2(slave):
   
    docker swarm join --token SWMTKN-1-2599d37taxeucuxqq0tbraz11h2e29h8upc1znl7ash4r8wzvg-0okv1lyptyeqcknmejrvthe0j      172.31.86.17:2377
      
   The output I got is:
   * nmejrvthe0j 172.31.86.17:
   * This node joined a swarm as a worker.
   
   4. Creating a service:
   
    docker service create --name service1 --publish published=80,target=80 --replicas 5 manilpuri9/portfolio

   5. Scaling my service  
   
   I. scaling up:
   
    docker service scale service1=10
   II. scaling down:
   
    docker service scale service1=3
   
   5. Rolling Updates:
   
    docker service update --image manilpuri9/portfolio:v1 service1
   
### b) Kubernetes Cluster
Kubernetes can perform everything that docker swarm performs but it outsmarts docker swarm:
* Kubernetes doesnot imediately kills the pods while rolling updates, First it checks if the newly created pod 
is stable or not, and check for loadbalencing and after that it removes the pod. 

Now I run my portfolio site in Kubernetes Cluster for getting all its benifits:
   
   1. Pulling image from my DockerHub Repo:
   
    docker pull manilpuri9/portfolio:v1
   
   2. Create namespace in kubernetes using "namespace.yml" file:
   
    kubectl create -f namespace.yml
   
   3. Create pods where there will be containers of portfolio site using "pod" file:
   
    kubectl -n NSmanil create -f pod
   
   4. Create an alias for ease of typing "kubectl -n manil" all the time 
   
    alias kctl="kubectl -n NSmanil"
   
   5. Create ReplicaController so that we can hirizontally scale at runtime and perform rolling updates:
   
    kctl expose --type=NodePort --port=80 rc/portfolio
   
   6. for horizontal scalling:
   
    kctl scale --replicas=15 rc/portfolio
   
   
   7. (Yet to perform)Similarly I can perform rolling updates at runtime if update my portfolio
   
    rolling-update --image=manilpuri9/portfolio:v2 manilpuri9/portfolio:v1
      
 ##  3 Intigration and Automation using Jenkins and Github
   
   1. Create an instance in AWS.
   
   2. Install Docker in it.
   
    yum install docker
    service docker start
   
   3. Pull jenkins image from dockerhub.
   
    docker pull jenkins
   
   4. Run Jinkins image.
   
    docker run -d -p 8080:8080 -p 50000:50000 jenkins
   
   5. Visit the public ip with port 8080, there is our jenkins running
   
    http://35.172.119.254:8080/
   
   6. follow the instruction to copy the password and make a user and login.
   
   7. install github plugins.
  
    https://wiki.jenkins.io/display/JENKINS/Plugins
   
   8. Create a simple job.
   
   9. Create a webhook from github repo
  
    https://www.youtube.com/watch?v=Z3S2gMBUkBo
   
   10. Test the github webhook.
   
   11. Add a jenkins slave to the job
   
    https://youtu.be/3AlR6bPcuvo
   
   12. Add a script in the job to download the github repo make a docker image and run it into slave.
   
   Script:
   
        git init
        git clone https://github.com/manilpuri9/manilpuri9.github.io
        yum install docker 
        service docker start
        docker build -t portfolio:latest .
        docker run -d -p 80:80 -v manilpuri9.github.io/:/var/www/html portfolio:latest
        
   13. Now whenever new commit is done, new image is made in the slave machine and it is hosted in the container at port 80.
   
## System configration and Deployment using Ansible
   Ansible is a configuration management tool similar to Chef, Puppet or Salt.
   I am using it for again hosting my portfolio site in AWS EC2 instance.
   
   1. I am installing ansible in my machine(OS: Ubuntu 16.04 LTS)
   
    apt-get install ansible
        
   2. Creating RSA key pair 
      
    ssh-keygen -t rsa
   
   3. While running this command rsa key pair is made in the folder /root/.ssh/
     
     cd /root/.ssh/
     ls
     
   Gives:
    
    id_rsa  id_rsa.pub  known_hosts

  I copy the id_rsa.pub in the servers .ssh file 
   4.
   5.
   
   
