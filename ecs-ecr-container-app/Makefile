export AWS_ACCESS_KEY_ID ?= test
export AWS_SECRET_ACCESS_KEY ?= test
export SERVICES = cloudformation,ecs,ec2
export AWS_DEFAULT_REGION = us-east-1
APP_PORT ?= 45139

usage:       ## Show this help
	@fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v fgrep | sed -e 's/\\$$//' | sed -e 's/##//'

install:     ## Install dependencies
	@which localstack || pip install localstack
	@which awslocal || pip install awscli-local

deploy:      ## Build and deploy the app locally
	@make install; \
		echo "Creating a new ECR repository locally"; \
		repoUri=$$(awslocal ecr create-repository --repository-name repo1 | jq -r '.repository.repositoryUri'); \
		echo "Building the Docker image, pushing it to ECR URL: $$repoUri"; \
		docker build -t "$$repoUri" .; \
		docker push "$$repoUri"; \
		docker rmi "$$repoUri"; \
		echo "Creating ECS infrastructure locally"; \
		awslocal cloudformation create-stack --stack-name infra --template-body file://templates/ecs.infra.yml; \
		echo "Deploying ECS app to local environment"; \
		awslocal cloudformation create-stack --stack-name sample \
			--template-body file://templates/ecs.sample.yml \
			--parameters ParameterKey=ImageUrl,ParameterValue=$$repoUri && \
		echo "ECS app successfully deployed. Sleeping a bit, then trying to access the app endpoint." && \
		sleep 5 && curl localhost:$(APP_PORT) && \
		echo "Sample app (nginx) successfully deployed. You can access it under http://localhost:$(APP_PORT)"

.PHONY: usage install deploy
