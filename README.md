# Hipster API Demo

## Install

_To resolve issues, please refer to *Toubleshooting* section below_

### Tool Prerequisites

`gcloud`
`kubectl`
`envsubst`

##### 

### APIs


#### Information Prerequisites
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

export ENDPOINTS_SERVICE_URL=hipstershop.endpoints.${PROJECT_ID}.cloud.goog

envsubst < endpoints/api_config-tmp.yaml > endpoints/api_config.yaml

gcloud endpoints services deploy endpoints/api_descriptor.pb endpoints/api_config.yaml

envsubst < deploy-manifests/* | kubectl apply -f -

# if the command above fails & returns *error: no objects passed to apply*), use this one:
for f in $(find deploy-manifests -regex '.*\.ya*ml'); do envsubst < $f | kubectl apply -f -; done;

kubectl apply -f istio-manifests

```
### Web App (using the REST APIs above)
```
envsubst < demo/hipster-web.yaml | kubectl apply -f -
```

## Test

### APIs

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

### Web App

The following comamnd
```
export WEB_URL=http://$(kubectl -n default get service hipster-web-external -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

curl $WEB_URL

#Or open the following URL in your browser:
echo $WEB_URL
```

## Troubleshooting

### (MacOS) bash: envsubst: command not found

`envsubst` is part of gettext. The quickest way [with brew](https://brew.sh/):
- `brew install gettext` (if not already installed)
- `brew link --force gettext` (if _symlink_ error, please see below)

### (MacOS) Could not symlink bin/autopoint

- ```sudo chown -R \`whoami`:admin /usr/local/share```
