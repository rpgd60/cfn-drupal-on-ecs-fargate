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
Project ?= drupal
ClusterName ?= drupal-ecs
EnvironmentName ?= dev
LogBucketName ?= logs-${ClusterName}-${AccountNumber}-${AwsRegion}
AwsRegion ?= eu-south-1  ## Using eu-south-1 for SSM Application Manager (CloudFormation Library) not available in Spain
Profile ?= sso-madmin
DrupalImage ?= drupal:latest
VpcCIDR ?= "10.200.0.0/16"
AccountNumber ?= $(shell aws sts get-caller-identity --profile sso-madmin --query Account --output text)

DrupalStackName ?= drupal-main-${AwsRegion}
InfraStackName ?= drupal-infra-${AwsRegion}
EfsStackName ?= drupal-efs

#######################################################
.PHONY: all
all:  drupal

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
			ClusterName=${ClusterName} \
			EnvironmentName=${EnvironmentName} \
			LogBucketName=${LogBucketName} \
		--no-fail-on-empty-changeset \
		--capabilities CAPABILITY_NAMED_IAM \
		--profile ${Profile} \
		--region ${AwsRegion}
.PHONY: drupal
drupal:
	aws cloudformation deploy \
		--template-file ./template.yaml \
		--stack-name ${DrupalStackName} \
		--parameter-overrides \
			InfraStackName=${InfraStackName} \
			ClusterName=${ClusterName} \
			EnvironmentName=${EnvironmentName} \
			LogBucketName=${LogBucketName} \
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
	@read -p "Are you sure that you want to destroy stack '${InfraStackName}'? [y/N]: " sure && [ $${sure:-N} = 'y' ]
	@echo "Deleting Logs Bucket"
	aws s3 rb s3://${LogBucketName} --force --region ${AwsRegion} --profile ${Profile} || true
	aws cloudformation delete-stack --region ${AwsRegion} --stack-name ${InfraStackName} --profile ${Profile}

