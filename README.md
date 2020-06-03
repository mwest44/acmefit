# acmefit
Setup Instructions for deploying ACME Fit Demo on TK cluster
Deploy ACMEFIT Application

Clone ACMEFIT REPO
	cd ~/demo-applications
	git clone https://github.com/vmwarecloudadvocacy/acme_fitness_demo.git

Connect to TK Cluster:

	kubectl vsphere login --server k8s.corp.local -u administrator@corp.local --tanzu-kubernetes-cluster-name tkg-cluster 
	--tanzu-kubernetes-cluster-namespace tkg --insecure-skip-tls-verify

	kubectl config use-context tkg-cluster
	kubectl get all


Create New Namespace and set context:
	kubectl create namespace acme-fit	 
	kubectl config set-context --current --namespace=acme-fit
	kubectl config view --minify | grep ‘namespace: acme-fit’

Enable RunAsRoot ClusterRole
	kubectl apply -f ~/demo-applications/allow-runasnonroot-clusterrole.yaml


Cart Service:

	Create Secret for Service to Auth to data cache
		kubectl create secret generic cart-redis-pass --from-literal=password=VMware1!

	Deploy Redis Cache and Cart Service:

		cd ~/demo-applications/acme_fitness_demo/kubernetes-manifests
		kubectl apply -f cart-redis-total.yaml
		kubectl apply -f cart-total.yaml


Catalog Service:


	Create Secret for Service to Auth to data cache
		kubectl create secret generic catalog-mongo-pass --from-literal=password=VMware1!

	Create Configmap to Initialize Mongo DB with catalog Items
		 kubectl create -f catalog-db-initdb-configmap.yaml

	Deploy Mongo Instance and Catalog Service
		kubectl apply -f catalog-db-total.yaml
		kubectl apply -f catalog-total.yaml


Payment Service:

	Create payment Service for the App
		kubectl apply -f payment-total.yaml

Order Service:

	Create Secret for Service to Auth to Orders Postgres DB
		kubectl create secret generic order-postgres-pass --from-literal=password=VMware1!

	Create Postgres DB instance and Order service
		kubectl apply -f order-db-total.yaml
		kubectl apply -f order-total.yaml


Users Service:

	Create Secret for Service to Auth to Users Datastore (Mongo) and Users cach (redis)
		kubectl create secret generic users-mongo-pass --from-literal=password=VMware1!
		kubectl create secret generic users-redis-pass --from-literal=password=VMware1!

	Create Configmap to Initialize Mongo DB with Initial Set of Users
		kubectl create -f users-db-initdb-configmap.yaml

	Deploy Users Database and Users Service
		kubectl apply -f users-db-total.yaml
		kubectl apply -f users-redis-total.yaml
		kubectl apply -f users-total.yaml	



Front End Service:

	Deploy Front End Service
		kubectl apply -f frontend-total.yaml

Point of Sale Service:

	Deploy Point of Sale Service:
		Edit point-of-sale-total.yaml and change service Type from NodePort to LoadBalancer
		Change the value of FRONTEND_HOST to frontend.acme-fit.svc.cluster.local
		kubectl apply -f point-of-sales-total.yaml



Basic Demo Flow


From the Homepage:

Frontend Service provides the UI and displays the products managed in the catalog service.

Click through a few products to see the catalog service written in Golang and stored in Mongo

Try to add an item to the cart.  Fail.  

Use the User service to authenticate

Now add a few items to the cart using the cart service and open the cart.

Place an order using the Order service backended by PostgresDB



Login as Username: eric         password:  vmware1!     And yes its all lower case
