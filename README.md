# Simple application for k8s deployment

Contains nodeserver with redis backend which displays a counter.


## Docker images

All the folders have their respective Dockerfile.

You can build the images using command

~~~~~~
	docker build -t="<image_name>" .

~~~~~


If you dont want to build up docker images from dockerfile then docker images are present on
dockerhub under the respectice repositries with versions latest and v1.

anandkarwa/k8snode1

anandkarwa/k8snode2

anandkarwa/k8sredis



## Deployment


kube_deploy.yml contains the deployment file.


Once you have your k8s cluster up and running just deploy the file using below command

~~~~~
	kubectl create namespace xor-poc
	kubectl create -f kube_deploy.yml

~~~~~


Check if all the pods are deployed

~~~~~
	kubectl get pods -n=xor-poc

~~~~~


Note: This may take few minutes on first deployment as docker images will be
	  pulled on each nodes.


Once all the pods are running then you see services running on port 32222 and 32111
To access http://<any-node-ip>:32111
and  http://<any-node-ip>:32222




# Commands for Rolling-updates and scaling

## Rolling update of image

## If you want to change the image name/version run following command

~~~~~~~~
	kubectl set image deployment/node1 node1=anandkarwa/k8snode1:v1 -n=xor-poc

~~~~~~~~


##If the deployment named node1's current size is 2 and you want to scale it to 3.

~~~~~
	kubectl scale --current-replicas=2 --replicas=3 deployment/node1 -n=xor-poc

~~~~~


## You can delete the pods via dashboard or by below command.

Check all the pods and their age

~~~~
	kubectl get pods -n=xor-poc

~~~~

Delete any pod of your choice

~~~~
	kubectl delete pod <pod-name>  -n=xor-poc

~~~~

Check the pods again there will be a new pod in state ContainerCreating or Running
in replacement for the deleted pod

~~~~~
	kubectl get pods -n=xor-poc

~~~~~


# Running pods on specific nodes

If you run the below command you will notice that your pods may be running on different nodes

~~~~~~
	kubectl get pods -o wide -n=xor-poc

~~~~~~

If you want your pods to run on specific node/nodes first we have to label our nodes

Run the following command to list down your nodes

~~~~~~
	kubectl get nodes

~~~~~~ 

Select one of the node (not master) to label it as node1 and the other as node2

Run the following command to achieve that

~~~~~~
	kubectl label nodes <nameof host1> type=node1
	kubectl label nodes <nameof host2> type=node2

~~~~~

To check the labels run

~~~~~
	kubectl get nodes --show-labels

~~~~~


kube_deploy_nodeselector.yml file is modified to select nodes with label node1 and node2
so that pods are provisioned as per labels

~~~~~
	kubectl apply -f kube_deploy_nodeselector.yml

~~~~~

Now if you check your pods again they will be running on the same hosts as per their lables

~~~~~~
	kubectl get pods -o wide -n=xor-poc

~~~~~~


## Using Images from Private Repository

# If you want to setup your own private repository on centos 

https://github.com/anandka/dockerbasic/blob/master/Docker-repo-setup.md


Once you have a private repository and its credentials you need to create secrets in order to use it.

To create secret run the below command also make sure your namespace is correct because secrets are't generic they are namespace specific.

~~~~~~
	# kubectl create secret docker-registry <secrets-name> --docker-server=<repo-server> --docker-username=<user-name> --docker-password=<password> --docker-email=<email-id> -n=<name-space>

	kubectl create secret docker-registry xorregistrykey --docker-server=xor.dockerrepo.com --docker-username=xor --docker-password=xoriant --docker-email=dummy@xor.com -n=xor-poc


~~~~~~


Now that you have created the secret we need to update the deployment file so as to point it to correct repository and provide the secret.

You have to now modify the image names and add a tag specifing your secretname in your deployment file.

To do it compare file kube_deploy_nodeselector.yml and kube_deploy_privaterepo.yml and edit it appropriately.

#Note:
	If your repo-server-name isnt configured in DNS and you are using self-signed certificates make sure you copy the certificates and extract them in your VM's also configure /etc/hosts file appropriately as mentioned in the setup document.