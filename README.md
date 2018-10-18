# Hipster API Demo

## Install 

#### Tool Prerequisites

`gcloud`
`kubectl` 
`envsusbt`
gcloud compute disks create --size=1GB --zone=us-central1-a istio-disk

sudo lsblk

sudo mkdir -p /mnt/disks/istio-disk

sudo mount -o discard,defaults /dev/sdb /mnt/disks/istio-disk

sudo chmod a+w /mnt/disks/istio-disk

gcloud scp endpoints/api_descriptor.pb <instance>:/mnt/disks/istio-disk

#### Information Prerequisite
Set your Google Cloud Project ID

```
export PROJECT_ID=<gcp-project-id>
export ZONE=us-central1-a	
```

Let's go

```
export CLUSTER_NAME=hipster-demo

gcloud config set project ${PROJECT_ID}

gcloud container clusters create ${CLUSTER_NAME} \
    --machine-type=n1-standard-2 \
    --num-nodes 3 \
    --enable-autoscaling --min-nodes 1 --max-nodes 10 \
    --cluster-version=1.10.7-gke.6 \
    --zone=${ZONE} \
    --no-enable-legacy-authorization

gcloud container clusters get-credentials ${CLUSTER_NAME} --zone=${ZONE}

kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user="$(gcloud config get-value core/account)"

kubectl apply -f istio-install/istio-demo.yaml

kubectl label namespace default istio-injection=enabled

kubectl apply -f deploy-manifests

kubectl apply -f istio-manifests

```

### Test

```
export GATEWAY_URL=http://$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

#Get List of Products
curl $GATEWAY_URL/products

#Get Ads
curl $GATEWAY_URL/ads

#Get Recommendations
curl $GATEWAY_URL/recommendations/1

#Get Currencies
curl $GATEWAY_URL/currencies

```
Explore the API endpoints in `endpoints/demo.http.swagger.json`

### Deploy webapp using the APIs
```
envsubst < demo/hipster-web.yaml | kubectl apply -f -
```