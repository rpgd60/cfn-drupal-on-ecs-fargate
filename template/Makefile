MAKEFILE_DIR := $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))
.DEFAULT_GOAL ?= help
.PHONY: help
help:
	@echo "${Project}"
	@echo "${Description}"
	@echo ""
	@echo " infra - deploy Bucket for Logs"
	@echo "	drupal - deploy (monolithic)"
	# @echo ${MAKEFILE_DIR}


###################### Parameters ######################
Description ?= Drupal on ECS-Fargate with EFS and Aurora Serverless v2
# CFN Stack Parameters
# Project ?= drupal
ProjectName ?= drupal-ecs
EnvironmentName ?= dev
LogBucketName ?= logs-${ProjectName}-${AccountNumber}-${AwsRegion}
## Using eu-south-1 for SSM Application Manager (CloudFormation Library) not available in Spain
Profile ?= sso-madmin
AwsRegion ?= eu-south-1
DrupalImage ?= drupal:latest
VpcCIDR ?= "10.200.0.0/16"
AccountNumber ?= $(shell aws sts get-caller-identity --profile sso-madmin --query Account --output text)
InstanceType ?= t4g.nano
DrupalStackName ?= drupal-main-${AwsRegion}
InfraStackName ?= drupal-infra-${AwsRegion}
VpcStackName ?= drupal-vpc-${AwsRegion}
DbStackName ?= drupal-db-${AwsRegion}
Ec2StackName ?= ec2-${AwsRegion}

## Ec2 Instances for tests
CreateUbuntu2204 ?= false
CreateUbuntu2404 ?= false
CreateAL2023 ?= true
#######################################################
.PHONY: ec2-efs
ec2-efs: infra vpc ec2 

.PHONY: ec2-db
ec2-db:  infra vpc ec2 db

.PHONY: debug-vars
debug-vars:
	@echo "Region: ${AwsRegion}"
	@echo "VPC CIDR:  ${VpcCIDR}"
	@echo "DrupalStackName: ${DrupalStackName}"
	@echo "Profile": ${Profile}

.PHONY: infra
infra:
	aws cloudformation deploy \
		--template-file ./infra.yaml \
		--stack-name ${InfraStackName} \
		--parameter-overrides \
			ProjectName=${ProjectName} \
			EnvironmentName=${EnvironmentName} \
			LogBucketName=${LogBucketName} \
			VpcCIDR=${VpcCIDR} \
		--no-fail-on-empty-changeset \
		--capabilities CAPABILITY_NAMED_IAM \
		--profile ${Profile} \
		--region ${AwsRegion}
.PHONY: vpc
vpc:
	aws cloudformation deploy \
		--template-file ./vpc-efs.yaml \
		--stack-name ${VpcStackName} \
		--parameter-overrides \
			InfraStackName=${InfraStackName} \
			ProjectName=${ProjectName} \
			EnvironmentName=${EnvironmentName} \
			LogBucketName=${LogBucketName} \
		--no-fail-on-empty-changeset \
		--capabilities CAPABILITY_NAMED_IAM \
		--profile ${Profile} \
		--region ${AwsRegion}
.PHONY: db
db:
	aws cloudformation deploy \
		--template-file ./db.yaml \
		--stack-name ${DbStackName} \
		--parameter-overrides \
			InfraStackName=${InfraStackName} \
			VpcStackName=${VpcStackName} \
			ProjectName=${ProjectName} \
			EnvironmentName=${EnvironmentName} \
		--no-fail-on-empty-changeset \
		--capabilities CAPABILITY_NAMED_IAM \
		--profile ${Profile} \
		--region ${AwsRegion}
.PHONY: drupal
drupal:
	aws cloudformation deploy \
		--template-file ./ecs_service.yaml \
		--stack-name ${DrupalStackName} \
		--parameter-overrides \
			InfraStackName=${InfraStackName} \
			VpcStackName=${VpcStackName} \
			DbStackName=${DbEfsStackName} \
			ProjectName=${ProjectName} \
			EnvironmentName=${EnvironmentName} \
			LogBucketName=${LogBucketName} \
		--no-fail-on-empty-changeset \
		--capabilities CAPABILITY_NAMED_IAM \
		--profile ${Profile} \
		--region ${AwsRegion}
.PHONY: ec2
ec2:
	aws cloudformation deploy \
		--template-file ./ec2.yaml \
		--stack-name ${Ec2StackName} \
		--parameter-overrides \
			InfraStackName=${InfraStackName} \
			VpcStackName=${VpcStackName} \
			DbStackName=${DbStackName} \
			ProjectName=${ProjectName} \
			InstanceType=$(InstanceType) \
			EnvironmentName=${EnvironmentName} \
			LogBucketName=${LogBucketName} \
			CreateAL2023=${CreateAL2023} \
			CreateUbuntu2204=${CreateUbuntu2204} \
			CreateUbuntu2404=${CreateUbuntu2404} \
		--no-fail-on-empty-changeset \
		--capabilities CAPABILITY_NAMED_IAM \
		--profile ${Profile} \
		--region ${AwsRegion}
.PHONY: del-drupal
del-drupal:
	@read -p "Are you sure that you want to destroy stack '${DrupalStackName}'? [y/N]: " sure && [ $${sure:-N} = 'y' ]
	aws cloudformation delete-stack --region ${AwsRegion} --stack-name ${DrupalStackName} --profile ${Profile}
.PHONY: del-infra
del-infra:
	@read -p "Are you sure that you want to destroy stack '${InfraStackName}'?  Will also delete Logs bucket '${LogBucketName}' [y/N]: " sure && [ $${sure:-N} = 'y' ]
	@echo "Deleting Logs Bucket"
	aws s3 rb s3://${LogBucketName} --force --region ${AwsRegion} --profile ${Profile} || true
	aws cloudformation delete-stack --region ${AwsRegion} --stack-name ${InfraStackName} --profile ${Profile}
.PHONY: del-infra
del-vpc:
	@read -p "Are you sure that you want to destroy stack '${VpcStackName}'? [y/N]: " sure && [ $${sure:-N} = 'y' ]
	aws cloudformation delete-stack --region ${AwsRegion} --stack-name ${VpcStackName} --profile ${Profile}

.PHONY: del-db-efs
del-db:
	@read -p "Are you sure that you want to destroy stack '${DbStackName}'? [y/N]: " sure && [ $${sure:-N} = 'y' ]
	aws cloudformation delete-stack --region ${AwsRegion} --stack-name ${DbStackName} --profile ${Profile}

.PHONY: del-ec2
del-ec2:
	@read -p "Are you sure that you want to destroy stack '${Ec2StackName}'? [y/N]: " sure && [ $${sure:-N} = 'y' ]
	aws cloudformation delete-stack --region ${AwsRegion} --stack-name ${Ec2StackName} --profile ${Profile}
