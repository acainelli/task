All masters comes with helm installed already, but for this task you gonna interact to your cluster from your local laptop.

### Download and install helm.

Helm is a binary, it means it's an executable file, you can either install it or download the binary ready for use, for your operating system. 
Official repository for download: https://github.com/helm/helm/releases/tag/v3.11.0
Ubuntu should be "linux-amd64". Unpack it and look for the "helm" binary


**INFO:** Everytime you download any binary, you have to move it to your executable path. 

Executable path = Every command is a file (ls, helm, mkdir, kubectl, nano, etc..) all of these commands is a file with scripts inside to perform something especific. Every operating system has 
different folders called executable path, this is the folder where your operating system will look for files to execute it. So if you download "helm" file, you can not run "helm ls" because linux 
won't recognize it as a command, unless it is inside a executable path folder. For linux there are some default folders, i.e:

		/usr/local/bin/
		/usr/bin/
		/bin/

So you gonna have to throw your helm binary to one of these folders in order to be executed from your terminal. The first 2 are the recommended ones.

Take any command as an example and you will find out where it is located with the command "which", That shows you where the specified command is located. "which vim", "which nano", "which tmate" 
(the one you installed to share terminal), etc .. 

### Testing helm installation

Test if your helm can be executed with `helm ls --all-namespaces`. If it returns a few releases or nothing it's working fine.
If it returns some connection refused error, means it's also fine, but you are not conneceted to any cluster.

_Error: Kubernetes cluster unreachable: Get "http://localhost:8080/version?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused_

You are not conneceted because you don't have any kubeconfig to connect to any cluster. So by default helm will look for kubernetes cluster installed locally (in your laptop, and you don't have 
kubernetes in your laptop, so it fails with connection refused).

So download the kubeconfig from Taikun and place it to the default folder with the name 'config' 

`~/.kube/config`

### Installing apps 
Once everything is done, you are ready to deploy applications using helm. For this example, we gonna deploy a nginx application from Bitnami repository.

- https://artifacthub.io/packages/helm/bitnami/nginx

In order to install the chart, you need to pass its repository. What does it means? if you want to install nginx, you must tell helm command where nginx chart is located. There is 2 ways of doing 
this.

1. You add the repository locally with **helm repo add _<name-you-want>_ _<repo-url>_**. i.e: `helm repo add my-test-repo https://charts.bitnami.com/bitnami`.
And then you can install telling **helm install _<name-you-want-for-your-release>_ _<repo-name-you-added>/<chart-name>_**. i.e: `helm install mynginx-1 my-test-repo/nginx`

2. If you don't want to waste time adding the repo, you can tell helm where to find the chart through the flag `--repo`, then you don't need do create name for your repo, only pass the url, followed 
by release-name and chart-name. i.e: `helm install --repo https://charts.bitnami.com/bitnami mynginx-1 nginx`

### 'Values' file

A helm chart can be customized by many ways, but it depends on what the creators of chart allowed you to customize. You can either change the app configuration or the kubernetes resources that 
deploys it. (Creating persistent volumes, different service types, etc...). Let's take a look to nginx default values.

- https://artifacthub.io/packages/helm/bitnami/nginx?modal=values

At the line 120, for example, we can see the parameter replicaCount with the default value 1. Means that if you install the chart and run `kubectl get po` you will see only 1 pod running for nginx 
from your release. There are 2 ways to customize it if you want. Let's suppose you want to have 3 replicas

**INFO:** The following examples tells you how to do it in case you haven't deployed your release yet. If you already did it, helm will return error saying that release my-nginx-1 already exists. 
So, If you have deployed already you can either create a new one my-nginx-2 or if you wanna keep using the same, you must update the current release. To do that just replace the following commands 
from `helm install ...` to `helm upgrade ...` instead. Also, if you decided to not add the repo, don't forget to pass `--repo`

1. Use the flag `--set` on helm command to set the value you want for some specific parameter. i.e: `helm install --set replicaCount=3 my-nginx-1 my-test-repo/nginx`

2. Create a yaml file with all customization you want and use `-f` to specify the file. i.e:

	create **values.yaml** with following content and save it.
	```
	replicaCount: 3
	```
	Tell helm you gonna use this files with your customizations. `helm install -f values.yaml my-nginx-1 my-test-repo/nginx`


The first option is used in case you want to pass just a few and short customizations. In case you have large changes in your charts, it's recommended to create the yaml file with everything and 
pass it to helm.

For example, going back to default values on artifacthub website, at the beginning of this step, let's go to line 517 and we will see that by default this chart will create a LoadBalancer as a 
service type. If we wanna change the type to NodePort along with the replicaCount, we can do it in both ways.

`helm install --set replicaCount=3 --set service.type=NodePort my-nginx-1 my-test-repo/nginx`

or, create **values.yaml**	
```
replicaCount: 3
service:
  type: NodePort
```

and the command will be the same we used in previous values.yaml example. `helm install -f values.yaml my-nginx-1 my-test-repo/nginx`

You can check the service type by running `kubectl get service`

