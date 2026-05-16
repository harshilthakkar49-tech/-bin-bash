#!/bin/bash

clear

read -p "Enter ZONE: " ZONE

export REGION="${ZONE%-*}"

gcloud compute networks create griffin-dev-vpc \
--subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp \
--network=griffin-dev-vpc \
--region $REGION \
--range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt \
--network=griffin-dev-vpc \
--region $REGION \
--range=192.168.32.0/20

gsutil cp -r gs://cloud-training/gsp321/dm .

cd dm

sed -i s/SET_REGION/$REGION/g prod-network.yaml

gcloud deployment-manager deployments create prod-network \
--config=prod-network.yaml

cd ..

gcloud compute instances create bastion \
--network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt \
--network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt \
--tags=ssh
