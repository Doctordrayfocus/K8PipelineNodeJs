VERSION 0.6
FROM bash:4.4
IMPORT ./templates/kubernetes AS nodejs_kubernetes_engine
IMPORT ./templates/docker AS nodejs_docker_engine
WORKDIR /build-arena

install:
	ARG service='sample'
	ARG envs='dev,prod'
	ARG version='0.1'
	ARG docker_registry='${DockerRegistry}'
	ARG apptype='nodejs'

	WORKDIR /setup-arena
	
	RUN mkdir $service $service/app

	COPY templates $service/templates
	COPY Earthfile $service

	FOR --sep="," env IN "$envs"	
		ENV dir="./$service/environments/$env"
		RUN echo "Creating environment $env"
	
		RUN mkdir -p $dir
		DO nodejs_kubernetes_engine+NODEJSAPP --service=$service --env=$env --dir=$dir --version=$version
	END

	SAVE ARTIFACT $service AS LOCAL ${service}

build:
	ARG version='0.1'
	ARG docker_registry='${DockerRegistry}'
	ARG service='sample'
	ARG envs='dev,prod'
	ARG node_env="developement"
	ARG apptype='nodejs'

	WORKDIR /build-arena

	COPY ./environments ${service}/environments
	
	BUILD nodejs_docker_engine+node-app --version=$version --docker_registry=$docker_registry --service=$service --node_env=$node_env


deploy:
	FROM alpine/doctl:1.22.2
	# setup kubectl
	ARG env='dev'
	ARG service='sample'
	ARG DIGITALOCEAN_ACCESS_TOKEN=""
	ARG apptype='nodejs'
	ARG version=""

	COPY ./environments ${service}/environments

	## Update apptemplate.yaml with latest versions

	DO nodejs_kubernetes_engine+NODEJSAPP --service=$service --version=$version --env=$envs 

	RUN kubectl version --client
	# doctl authenticating
    RUN doctl auth init --access-token ${DIGITALOCEAN_ACCESS_TOKEN}

	# save Kube config
	RUN doctl kubernetes cluster kubeconfig save cluster-name
	RUN kubectl config get-contexts	

	## deploy kubernetes configs
	RUN kubectl cp $service/environments/${envs}/extras-$service $(kubectl get pod -l app=pipelineapptemplate-controller -o jsonpath="{.items[0].metadata.name}"):/usr/src/app/configs
	RUN kubectl apply -f $service/environments/${envs}/app-template.yaml

auto-deploy:
	ARG version='0.1'
	ARG docker_registry='${DockerRegistry}'
	ARG service='sample'
	ARG env='dev'
	ARG apptype='nodejs'

	# Build and push docker images
	BUILD +build

	# Deploy to kubernetes
	BUILD +deploy

	




