VERSION 0.6
FROM bash:4.4
IMPORT ./templates/kubernetes AS nodejs_kubernetes_engine
IMPORT ./templates/docker AS nodejs_docker_engine
WORKDIR /build-arena

install:
	ARG service='sample'
	ARG envs='dev,prod'
	ARG version='0.1'
	ARG docker_registry='docker.io'
	ARG apptype='nodejs'
	ARG upload_url="http://localhost:8080/save-service-setup"

	RUN apk add zip

	RUN apk add curl

	WORKDIR /setup-arena
	
	RUN mkdir $service $service/app

	COPY templates $service/templates
	COPY Earthfile $service

	FOR --sep="," env IN "$envs"	
		ENV dir="./$service/environments/$env"
		RUN echo "Creating environment $env"
	
		RUN mkdir -p $dir $dir/extras-$service
		DO nodejs_kubernetes_engine+NODEJSAPP --service=$service --env=$env --dir=$dir --version=$version
		DO nodejs_kubernetes_engine+CONFIGMAP --service=$service --env=$env 
		DO nodejs_kubernetes_engine+SECRETS --service=$service --env=$env
	END

	RUN zip -r  /setup-arena/${service}.zip /setup-arena/${service}

	RUN curl -F "data=@/setup-arena/$service.zip" ${upload_url}


build:
	ARG version='0.1'
	ARG docker_registry='docker.io'
	ARG service='sample'
	ARG env='dev,prod'
	ARG node_env="developement"
	ARG apptype='nodejs'

	WORKDIR /build-arena

	COPY ./environments ${service}/environments
	
	BUILD nodejs_docker_engine+node-app --version=$version --docker_registry=$docker_registry --service=$service --node_env=$node_env


deploy:
	FROM alpine/doctl:1.22.2
	# setup kubectl
	ARG DIGITALOCEAN_ACCESS_TOKEN=""
	ARG service=""
	ARG env=""
	ARG version=""
	ARG template=""

	COPY ./environments ${service}/environments

	## Update apptemplate.yaml with latest versions

	DO nodejs_kubernetes_engine+NODEJSAPP --template=$template --service=$service --envs=$env

	RUN kubectl version --client
	# doctl authenticating
    RUN doctl auth init --access-token ${DIGITALOCEAN_ACCESS_TOKEN}

	# save Kube config
	RUN doctl kubernetes cluster kubeconfig save cluster-name
	RUN kubectl config get-contexts	

	## deploy kubernetes configs
	RUN kubectl cp $service/environments/${env}/extras-$service $(kubectl get pod -l app=pipelineapptemplate-controller -o jsonpath="{.items[0].metadata.name}"):/usr/src/app/configs
	RUN kubectl apply -f $service/environments/${env}/app-template.yaml

