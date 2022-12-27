<h1 align="center">
  kubernetes 101 for developers
</h1>

> This is a hands-on tutorial to deploy basic services with kubectl and helm on kubernetes cluster running on local machine.

👉🏼 No cloud account needed.

👉🏼 No Docker desktop needed.

## Table of contents
---
- [System Setup](#system-setup)
- [kubectl](#kubectl)
	- [Get ready](#get-ready)
	- [Create a Deployment](#create-a-deployment)
	- [Create a Service](#create-a-service)
	- [Clean up](#clean-up)
- [kustomize](#kustomize)
	- [Get ready](#get-ready-1)
	- [Deploy apps](#deploy-apps)
- [HELM](#helm)
- [Argo CD](#argo-cd)
- [TO-DO](#to-do)

## System Setup
---
1.  install [Docker CLI](https://minikube.sigs.k8s.io/docs/tutorials/docker_desktop_replacement/)

	```shell
	brew install docker
	```

2.  install [hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/) to be used as docker run time.

	```shell
	brew install hyperkit
	```

3.  install [minikube](brew install minikube
    ) to be used as kubernetes cluster

	```shell
	brew install minikube
	```

4.  configure minikube resources (optional)

	```shell
	minikube config set cpus 6
	minikube config set memory 12g
	```

5.  start minikube

	```shell
	minikube start  --driver=hyperkit --container-runtime=docker
	```

	![minikube start](./docs/minikube_start.png)

6.  verify minikube

	```shell
	minikube kubectl get nodes
	```

	!minkube get pods](./docs/minikube_get_po.png)

7.  point terminal's Docker CLI to the Docker instance inside minikube

	```shell
	eval $(minikube docker-env)
	```

8.  Optionally you can open minikube dashboard

	```shell
	minikube dashboard
	```

	![minikube dashboard](./docs/minikube_dashboard.png)

	It should open a dashboard UI in browser like follow:
	
	![minikube dashboard UI](./docs/dashboard_demo.png)

## kubectl
---
### Get ready

1. start minikube

	```shell
	minikube start  --driver=hyperkit --container-runtime=docker
	```

2. point terminal's Docker CLI to the Docker instance inside minikube

	```shell
	eval $(minikube docker-env)
	```

3. build demo app image

	```shell
	cd apps/demo-api
	docker build -t demo-api .
	```
	
	![Build demo](./docs/build_demo_api.png)

### Create a Deployment


1. Create demo app Deployment

	Create a Deployment that manages a Pod. The Pod runs a Container based on the provided Docker image.
	We can not set image pull policy on CLI and we want to use the local image, so we will create a simple config file to from CLI itself and add the parameter for image pull policy to "Never".

	**Create a deployment file.**

	```shell
	cd ../ # if not on the root path
	cd kubectl
	```

	```shell
	kubectl create deployment demo-api --image=demo-api:latest --dry-run=client -o yaml > deployment.yaml
	```
	
	![build demo api](./docs/build_demo_api.png)
	
	Now Modify `spec.template.spec.containers` in deployment.yaml created.
	
	```shell
	spec:
		containers:
		- image: demo-api:latest
		name: demo-api
		resources: {}
	```
  
	and add `imagePullPolicy: Never` so that it looks like:
    
	```shell
	spec:
		containers:
		- image: demo-api:latest
		name: demo-api
		resources: {}
		imagePullPolicy: Never
	```

	**Apply the config file with kubectl**
	
	```shell
	kubectl apply -f deployment.yaml
	```
	
	Output is similar to:
	
	```shell
	kubectl apply -f deployment.yaml
	deployment.apps/demo-api created
	```


2. View the Deployment:

   ```shell
   kubectl get deployments
   ```

   The output is similar to:

   ```
   NAME       READY   UP-TO-DATE   AVAILABLE   AGE
   demo-api   1/1     1            1           28s
   ```

3. View the Pod:

	```shell
	kubectl get pods
	```

	The output is similar to:

	```shell
	NAME                        READY   STATUS    RESTARTS   AGE
	demo-api-7c7d9f4689-sbkns   1/1     Running   0          100s
	```

4. View cluster events:

	```shell
	kubectl get events
	```

5. View the `kubectl` configuration:

	```shell
	kubectl config view
	```

### Create a Service


By default, the Pod is only accessible by its internal IP address within the
Kubernetes cluster. To make the `demo-api` Container accessible from outside.

1. Expose the Pod to the public internet using the `kubectl expose` command:

	```shell
	kubectl expose deployment demo-api --type=LoadBalancer --port=3000
	```

2. View the Service you created:

	```shell
	kubectl get services
	```

	The output is similar to:

	```shell
	NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
	demo-api     LoadBalancer   10.101.236.39   <pending>     3000:30490/TCP   13s
	kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          21s
	```

3. Run the following command to open the app:

	```shell
	minikube service demo-api
	```

	The output is similar to:

	```shell
	|-----------|----------|-------------|----------------------------|
	| NAMESPACE |   NAME   | TARGET PORT |            URL             |
	|-----------|----------|-------------|----------------------------|
	| default   | demo-api |        3000 | http://192.168.106.3:30490 |
	|-----------|----------|-------------|----------------------------|
	🎉  Opening service default/demo-api in default browser...
	```

	It should open the following page in the browser:

	![express app demo](./docs/express_demo_api.png)

### Clean up

1. Now you can clean up the resources you created in your cluster:

	```shell
	kubectl delete service demo-api
	kubectl delete deployment demo-api
	```

2. Optionally, stop the Minikube virtual machine (VM):

	```shell
	minikube stop
	```

3. Optionally, delete the Minikube VM:

	```shell
	minikube delete
	```

## kustomize
---
In this we will deploy multiple services. To do so we will be using multiple config files for kubernetes and will manage the config with help of [kustomize](https://github.com/kubernetes-sigs/kustomize). This will deploy a proper backend, frontend and db instance to power up a full application stack and use it as we do in production. This example will use the realworld blog app API and UI example apps.

![kustomize deployment diagram](./docs/k8s_diagram.png)

We will deploy following services:

1. blog-api: expressjs backend REST API server.
2. blog-ui: react UI app being served by a nginx server.
3. postgres: db for blog-api backend service.

### Get ready

1. start minikube

	```shell
	minikube start  --driver=hyperkit --container-runtime=docker
	```

2. point terminal's Docker CLI to the Docker instance inside minikube

	```shell
	eval $(minikube docker-env)
	```

3. build blog-api app image

	```shell
	cd apps/realblog/blog-api
	docker build -t blog-api .
	```

	Output will be similar to:

	```shell
	~/dev/learn/kubernetes  cd apps/realblog/blog-api
	docker build -t blog-api .
	Sending build context to Docker daemon  2.048kB
	Step 1/10 : FROM node:16.15.0
	---> 9d200cd667d5
	Step 2/10 : RUN mkdir -p /home/node/app && chown -R node:node /home/node/app
	---> Using cache
	---> 48145b89be5e
	Step 3/10 : WORKDIR /home/node
	---> Using cache
	---> 280c540f8d5c
	Step 4/10 : RUN git clone --depth 1 https://github.com/tanem/express-bookshelf-realworld-example-app.git app
	---> Using cache
	---> 34dd1b5c6ded
	Step 5/10 : RUN chown -R node:node /home/node/app
	---> Using cache
	---> d95cb606a0de
	Step 6/10 : WORKDIR /home/node/app
	---> Using cache
	---> 740cacc09bdd
	Step 7/10 : RUN npm i
	---> Using cache
	---> 7b86ce450a87
	Step 8/10 : EXPOSE 3000
	---> Using cache
	---> 1b4d3ba26c7b
	Step 9/10 : USER node
	---> Using cache
	---> 06c0b457b04a
	Step 10/10 : CMD [ "./bin/start.sh" ]
	---> Using cache
	---> 487c135295f6
	Successfully built 487c135295f6
	Successfully tagged blog-api:latest
	``
	```

4. build blog-ui app image

	```shell
	cd apps/realblog/blog-ui
	docker build -t blog-ui .
	```

	Output will be similar to: 

	```shell
	~ cd apps/realblog/blog-ui
	~ docker build -t blog-ui .
	Sending build context to Docker daemon   2.56kB
	Step 1/11 : FROM node:14.19.3 as base
	---> f3ec39298d1b
	Step 2/11 : RUN mkdir -p /home/node/app && chown -R node:node /home/node/app
	---> Using cache
	---> 49cc9653ba42
	Step 3/11 : WORKDIR /home/node
	---> Using cache
	---> 8668cbb0d4c8
	Step 4/11 : RUN git clone --depth 1 https://github.com/angelguzmaning/ts-redux-react-realworld-example-app.git app
	---> Using cache
	---> 24326deaa442
	Step 5/11 : RUN chown -R node:node /home/node/app
	---> Using cache
	---> f0a11bcbee98
	Step 6/11 : WORKDIR /home/node/app
	---> Using cache
	---> 7b2bb3a27151
	Step 7/11 : RUN npm i
	---> Using cache
	---> bd859e464065
	Step 8/11 : RUN npm run build
	---> Using cache
	---> f2249c73479a
	Step 9/11 : USER node
	---> Using cache
	---> 9a1326890a11
	Step 10/11 : FROM nginx
	---> 1403e55ab369
	Step 11/11 : COPY --from=base /home/node/app/build /usr/share/nginx/html
	---> Using cache
	---> 29b831cc3064
	Successfully built 29b831cc3064
	Successfully tagged blog-ui:latest
	```

5. enable ingress [plugin](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)

	Ref: https://kubernetes.io/docs/concepts/services-networking/ingress/

	```shell
	minikube addons enable ingress
	```

6. add domain to hosts
	
	**Get minikube IP**
	```shell
	minikube ip
	```
	**update your machine's hosts file**
	```shell
	sudo vi /etc/hosts
	```
	Add the host `realblog.local` to the file and point it to minikube IP. 

	Hosts should look like: 

	```shell
	127.0.0.1	localhost
	255.255.255.255	broadcasthost
	::1             localhost

	192.168.106.3 realblog.local
	```


### Deploy apps

1. apply config with kustomize to kubectl

	```shell
	cd kustomize/realblog
	kubectl apply -k ./
	```
	Output will be similar to:
	
	```shell
	service/realblog-api created
	service/realblog-postgres created
	service/realblog-ui created
	persistentvolumeclaim/postgres-pv-claim created
	deployment.apps/realblog-api created
	deployment.apps/realblog-postgres created
	deployment.apps/realblog-ui created
	ingress.networking.k8s.io/realblog-ingress created
	```

2. verify deployment and services

	list deployments
	```shell
	kubectl get deployments
	```

	Output will be similar to:

	```shell
	kubectl get deployments
	NAME                READY   UP-TO-DATE   AVAILABLE   AGE
	realblog-api        1/1     1            1           2m44s
	realblog-postgres   1/1     1            1           2m44s
	realblog-ui         1/1     1            1           2m44s
	```

	list pods
	```shell
	kubectl get pods
	```

	Output will be similar to:

	```shell
	NAME                                 READY   STATUS    RESTARTS   AGE
	realblog-api-785df47759-plwjw        1/1     Running   0          4m15s
	realblog-postgres-5967b7666c-5t8c4   1/1     Running   0          4m15s
	realblog-ui-6fc444cb95-zqcjt         1/1     Running   0          4m15s	```

	list services
	```shell
	kubectl get services
	```

	Output will be similar to:

	```shell
	NAME                TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
	kubernetes          ClusterIP      10.96.0.1      <none>        443/TCP          12h
	realblog-api        LoadBalancer   10.99.28.2     <pending>     3000:30148/TCP   5m23s
	realblog-postgres   ClusterIP      None           <none>        5432/TCP         5m23s
	realblog-ui         LoadBalancer   10.99.96.128   <pending>     80:32187/TCP     5m23s
	```

	list ingress
	```shell
	kubectl get ingress
	```

	Output will be similar to:

	```shell
	NAME               CLASS   HOSTS            ADDRESS         PORTS   AGE
	realblog-ingress   nginx   realblog.local   192.168.106.3   80      6m21s
	```

3. Use blog app

	open "http://realblog.local" in browser.
	A blogger site should open. Create a user and add some articles to test the app follow. 

	![blog home](./docs/blog_home.png)

### Clean up

1. Now you can clean up the resources you created in your cluster:

	Run the following command from `kustomize/realblog`:

	```shell
	kubectl delete -k ./
	```

2. Optionally, stop the Minikube virtual machine (VM):

	```shell
	minikube stop
	```

3. Optionally, delete the Minikube VM:

	```shell
	minikube delete
	```

## [HELM](https://helm.sh/docs/)

### Get ready

1. start minikube

	```shell
	minikube start  --driver=hyperkit --container-runtime=docker
	```

2. point terminal's Docker CLI to the Docker instance inside minikube

	```shell
	eval $(minikube docker-env)
	```

3. build demo-api app image

	```shell
	cd apps/demo-api
	docker build -t demo-api .
	```


remove helm/realblog : rm -rf helm/realblog
cd helm
helm create realblog
10432  helm install realblog
10433  ls
10434  cd realblog
10435  helm install realblog .
10436  helm upgrade realblog .
10437  eval $(minikube docker-env)
10438  helm upgrade realblog .
10439  k get po

in helm/realblog/values.yaml

update `service.port` to 3000 , `image.repository` to "demo-api" `image.tag` to latest

helm upgrade realblog . 

follow on screen instructions  or kubectl --namespace default port-forward deployment/realblog 8080:3000

a page should open like follow


kubectl --namespace default port-forward deployment/realblog-realblog-api 80

ingress: update values.yaml.ingress.hosts.host ingress.enabled: true

## [Argo CD](https://argo-cd.readthedocs.io/en/stable/?_gl=1*iazngm*_ga*MjE4MzA1OTYwLjE2NzIxMzMyNTg.*_ga_5Z1VTPDL73*MTY3MjEzMzI1Ny4xLjAuMTY3MjEzMzI1Ny4wLjAuMA..)


## TO-DO

- Add helm charts flow
- Add argo CD flow
