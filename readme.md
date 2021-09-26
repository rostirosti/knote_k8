## 

## Application is from the excellent guide at https://learnk8s.io/nodejs-kubernetes-guide 

# Option 1 test on local


- Make sure you have docker on your machine
- Go to app root directory.  Edit newrelic.js to your liking.  Leave the license key blank (we will send that part through an environmental variable)
- Package it up as a docker image.  Execute the following "dockebuild -t knote ." 
- Create a Docker network so that Node and MongoDB can communicate with each other "docker network create knote"
- Now lets run the application.  For this to work we need to first fire up the Mongo DB container and then the Node container.
- Run the following "docker run --name=mongo --rm --network=knote mongo"
- Now lets run our application and load up the environmental variable for the New Relic license key.  Make sure to replace the indicated portion with your license key: docker run  -e NEW_RELIC_LICENSE_KEY=REPLACEWITHYOURKEY --name=knote --rm --network=knote -p 3000:3000 -e MONGO_URL=mongodb://mongo:27017/dev knote
- You should see the new app by opening your browser and going to http://localhost:3000

# Option 2 - Skip a step and quickly get the app up and running on Kuberenetes

- We will be using the docker image from my container registry here
- Copy the files from the "kube" directory and load them in your text editor
- In knote_deployment.yml replace the NEW_RELIC_LICENSE_KEY value with your license key
- If you are deploying your application on a single VM server (IE you deploying on a single node cluster on something like an EC2 instance) then make sure to use the knote_services_vm.yml otherwise for a managed Kubernetes service you should be able to use knote_services_managed,.yml.  
- You should have only one knote_services file in your directory so figure out which one you will use and erase the other one.  The difference is really only the "externalIPs" setting which needs to contain the external IP of your VM.   
- Once you are all ready to deploy load up all the YAML files in a directory on your server then run "kubectl apply -f knote_yaml"  knote_yaml should be replaced with the name of the folder which contains all your YAML files.  
- Run "kubectl get pods --all-namespaces" to verify that all your pods are up
- Check the IP of the load balancer by running "kubectl get services" 

# Option 3 Docker w/No APM on Ubuntu 18.4

- install docker using the snap store
sudo snap install docker

- test docker  
docker run hello-world

- make sure you can see 
"Hello from Docker!
This message shows that your installation appears to be working correctly.""

- clone this repo https://github.com/rostirosti/knote_k8
git clone https://github.com/rostirosti/knote_k8

- navigate into the knote_k8 folder 

- package up the contents in a docker image
docker build -t knote .

- congrats you just created a docker image with your app!
#now lets create a network for our docker container so that the Node and MongoDB can communicate to each other
docker network create knote 

- now open up two more tabs or terminal sessions/windows. ssh into the same server on all of the tabs/windows.

- now on one of them run 
docker run --name=mongo --rm --network=knote mongo

- switch to the second tab and run
docker run --name=knote --rm --network=knote -p 3000:3000 -e MONGO_URL=mongodb://mongo:27017/dev knote

- navigate into your app (ip address of your box and :3000 Example: http://144.202.18.135:3000)

- you should see your app! add some notes!

- now go to New Relic "Add More Data" > Select Guided Install and deploy the Linux Infra agent

- install the logs integration and the golden signals integration - do not deploy the mongo integration

- during the installation it will find some logs to tail, leave those selected and proceed

- you will see that we are not picking up any docker logs - lets fix that!
- first we need to find where they are stored (hint: /var/snap/docker/common/var-lib-docker/containers)

- a few containers in here !!! each of these folders contains a .log file. as more are created you can bet that there would be more folders. would be really annoying to create so many log configurations! can we use wildcards?

- check the docs https://docs.newrelic.com/docs/logs/enable-log-management-new-relic/enable-log-monitoring-new-relic/forward-your-logs-using-infrastructure-agent/ and apply changes to the logging.yml config in /etc/newrelic-infra/logging.d/ folder using the path to our logs /var/snap/docker/common/var-lib-docker/container and wildcards in the path

- also take a minute and add an extra attribute to your logs "logtype:dockerstuff"

- IMPORTANT - YAML is very sensitive to cases, spaces, and so forth so be careful of that 

- hint there is an example on that docs page under """ - just make sure you modify the path

- answer:     file: /var/snap/docker/common/var-lib-docker/containers/*/*.log
Log into New Relic and check the logs that we are getting.  If you do not see any logs try killing your docker containers by running the ""docker kill knote" command and running through the following again 

- now on one of them run 
docker run --name=mongo --rm --network=knote mongo

- switch to the second tab and run
docker run --name=knote --rm --network=knote -p 3000:3000 

-e MONGO_URL=mongodb://mongo:27017/dev knote

#if you still do not see logs go and reach out for support in the slack channel

#next:
parsing logs in UI
data dropping rules





