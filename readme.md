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


