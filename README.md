# Hipster API Demo

## Install 

#### Tool Prerequisites

`gcloud`
`kubectl` 
`envsusbt`

#### Information Prerequisite
Set your Google Cloud Project ID

```
export PROJECT_ID=<gcp-project-id>
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
    --no-enable-legacy-authorization

gcloud container clusters get-credentials ${CLUSTER_NAME}

kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user="$(gcloud config get-value core/account)"

kubectl apply -f istio-install/istio-demo.yaml

kubectl label namespace default istio-injection=enabled

export ENDPOINTS_SERVICE_URL=hipstershop.endpoints.${PROJECT_ID}.cloud.goog

envsubst < endpoints/api_config-tmp.yaml > endpoints/api_config.yaml

gcloud endpoints services deploy endpoints/api_descriptor.pb endpoints/api_config.yaml

envsubst < deploy-manifests/* | kubectl apply -f -

kubectl apply -f istio-manifests

```

### Test

```
export GATEWAY_URL=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

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

