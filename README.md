# teamscale-helm

Prerequisites:
- Download the oc client
- Download helm

## How to install Helm charts?
1. Login to oc (token can be retrieved from app)
> oc login --token=&lt;token&gt; --server=https://api.sandbox.x8i5.p1.openshiftapps.com:6443
2. Set TILLER_NAMESPACE as environment variable
> set TILLER_NAMESPACE=&lt;your-namespace&gt;
3. Start tiller container
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


## How to add your docker login to pull images?
1. Create a new secret
> oc create secret docker-registry &lt;secret-name&gt; --docker-server=docker.io --docker-username=&lt;your-username&gt; --docker-password=&lt;your-access-token&gt; --docker-email=&lt;your-email&gt;
2. Link your secret to the default service account to pull images
> oc secrets link default &lt;secret-name&gt; --for=pull