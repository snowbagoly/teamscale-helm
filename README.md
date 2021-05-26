# teamscale-helm

Prerequisites:
- Download the oc client
- Download helm

## How to install Helm charts?
1. Login to oc (token can be retrieved from app clicking on the ? icon > Command line tools > Copy Login Command)
> oc login --token=&lt;token&gt; --server=https://api.sandbox.x8i5.p1.openshiftapps.com:6443
2. Set TILLER_NAMESPACE as environment variable
> set TILLER_NAMESPACE=&lt;your-namespace&gt;
3. Start tiller container if not yet running
> oc process -f https://github.com/openshift/origin/raw/master/examples/helm/tiller-template.yaml -p TILLER_NAMESPACE=&lt;your-namespace&gt; -p HELM_VERSION=v2.9.0 | oc create -f -
4. Check tiller deployment status
> oc rollout status deployment tiller
5. Start helm
> helm init --stable-repo-url https://charts.helm.sh/stable --service-account tiller --client-only
6. Check helm version to see that it could connect
> helm version
7. Add some permissions to your user
> oc policy add-role-to-user edit "system:serviceaccount:&lt;your-namespace&gt;:tiller"
8. Install your app with helm
> helm install &lt;path-to-your-helmcharts-can-be-local-or-url&gt; -n &lt;arbitrary-release-name&gt;


## How to debug Helm charts?
1. Login to oc and set up helm as described in **How to install Helm charts**
2. Run the following command to print all generated files:
> helm install &lt;path-to-your-helmcharts-can-be-local-or-url&gt; --dry-run --debug


## How to clean up a previous release?
1. Login to oc and set up helm
2. Look for the name of your previous release (a ConfigMap is created with the corresponding release name)
3. Run the following command:
> helm del --purge &lt;your-release-name&gt;
4. Wait until everything is cleaned up (for me the deployment had to be deleted manually, otherwise the PVCs were also not cleaned up)

## How to update an existing release?
Not sure that this will be possible with ARA, but it would make easier to apply small changes (e.g. patch update or change routes).
0. Make your changes
1. Login to oc and set up helm
2. helm upgrade --install &lt;your-release-name&gt; &lt;path-to-your-helmcharts-can-be-local-or-url&gt;

**Some of the fields (e.g. host name in route) are immutable, cannot be updated. To update them, you have to change the name of the object: this will result in deleting the previous object and creating a new one, so you have to be careful not to lose data. Once the update also set my PVCs to Terminating state (when changing my deployment name), which sounds a bit risky as they were just deleted when my previous pod scaled down to 0 - even though my new deployment wanted to use them.**

## How to add your docker login to pull images?
1. Create a new secret
> oc create secret docker-registry &lt;secret-name&gt; --docker-server=docker.io --docker-username=&lt;your-username&gt; --docker-password=&lt;your-access-token&gt; --docker-email=&lt;your-email&gt;
2. Link your secret to the default service account to pull images
> oc secrets link default &lt;secret-name&gt; --for=pull