# Cloud Identity Demo
This documentation provides details for how to configure and setup GCP Apigee & Cloud Identity Platform with the Hipster App.

* [Prerequisites](#prereqs)
  * [Setup GKE](#setup_gke)
  * [Install Istio](#install_istio)
  * [Install Hipster App](#install_hipster_app)
* [Apigee Adpater](#apigee_adpater)
  * [Install Apigee Adapter](#install_apigee_adapter)


## <a name="prereqs">Prerequisites</a>
These are the prerequisites for the Controller App Development


### <a name="setup_gke">Setup GKE</a>
Run through all commands up to [Setup GKE](../README.md#setup-gke) for setting up the GKE environment and cluster.


### <a name="install_istio">Install Istio</a>
Run through all commands up to and including [Install Istio](../README.md#install-istio) for setting up the Istio environment and cluster. Do not apply the *install/kubernetes/istio-demo.yaml* installation though since it does not include mTLS. If you accidentally apply it, you can always apply the next command over it:

Apply the Istio mTLS configuration instead of the default instance:

		kubectl apply -f install/kubernetes/istio-demo-auth.yaml

		cd ..

Enable sidecar injection:

		kubectl label namespace default istio-injection=enabled

Apply the mTLS mesh policy, virtual servies, mTLS destination rules, frontend ingress gateway, and whitelist for external apis:

		kubectl apply -f istio-manifests/networking/

Verify everything was installed:

		kubectl get meshpolicy,virtualservice,destinationrule,serviceentry

Apply the Auth policy, service role, and service role binding to enforce RBAC on the checkout services:

		kubectl apply -f istio-manifests/rbac/

Verify everything was installed:

		kubectl get policies,serviceroles,servicerolebinding


### <a name="install_hipster_app">Install Hipster App</a>
Install the application:

		kubectl apply -f kubernetes-manifests/hipster-app.yaml

Verify everything is running:
_takes about a minute or two_

		watch kubectl get pods

Export FRONTEND_URL to navigate to Hipster App frontend:

	export FRONTEND_URL=http://$(kubectl get service frontend-external -o jsonpath='{.status.loadBalancer.ingress[0].ip}'); echo $FRONTEND_URL

Launch the FRONTEND_URL in a modern browser (Chrome/Safari/Firefox) and navigate around the Hipster Shop


### <a name="install_apigee_adapter">Install Apigee Adapter</a>
Run through the download commands for the [Install Apigee Adapter](../README.md#install-apigee-adapter) for setting up the Apigee Istio Mixer adapter. Save the *handler.yaml* file into this [directory](../istio-manifests/apigee/)

Apply all the Apigee Mixer configurations:

		kubectl apply -f istio-manifests/apigee/apigee-adapter.yaml
		kubectl apply -f istio-manifests/apigee/definitions.yaml
		kubectl apply -f istio-manifests/apigee/handler.yaml

Apply the Apigee mixer rule:

		kubectl apply -f istio-manifests/apigee/rule.yaml

You will now need a valid Application and API Key to connect to the Hipster App services

