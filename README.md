## nginx-operator
Trivial helm-based nginx operator

## Controller
Controllers inside the cluster detect the requested changes and take action to adjust the state. This occurs asynchronously, after you’ve submitted your new manifests.

## Operator
Controller + API + CRD

The operator concept provides a way to monitor these applications using the same principles as regular controllers. An operator is simply a specialized controller that uses custom resources to move a specific application into the correct user-defined state.

Operators make it easier to install, configure, and maintain the managed software in your cluster.

3 types of operators:
- Anisble-bases (simple)
- Helm-based (simple)
- go-based (complex)

## Helm-based Operator
https://sdk.operatorframework.io/docs/building-operators/helm/tutorial/

- cd /nginx-operator

Using custom Helm Chart:
- `operator-sdk init --plugins helm --domain jkosik --group webserver --version v1alpha1 --kind Nginx` results in APIVersion `webserver.jkosik/v1alpha1` and Kind Nginx.

Using existing Helm Chart:
- `operator-sdk init --plugins helm --domain jkosik --helm-chart bitnami/nginx` results in APIVersion `charts.jkosik/v1alpha1`

- update `config/rbac/role.yaml` if needed

### Build and Operator's image
- update `Makefile`:
-IMG ?= controller:latest
+IMG ?= $(IMAGE_TAG_BASE):$(VERSION)

- `make docker-build docker-push` => `docker.io/jkosik/nginx-operator:0.0.1`

### Deploy Operator
- Operator can be (1) deployed outside of the cluster, (2) in-cluster as Deployment or (3) using OLM(Operator Lifecycle Manager) using bundles.

#### (2) In-cluster deployment
```
❯ make deploy
cd config/manager && /Users/juraj/data/github/nginx-operator/bin/kustomize edit set image controller=jkosik/nginx-operator:0.0.1
/Users/juraj/data/github/nginx-operator/bin/kustomize build config/default | kubectl apply -f -
namespace/nginx-operator-system created
customresourcedefinition.apiextensions.k8s.io/nginxes.charts.jkosik created
serviceaccount/nginx-operator-controller-manager created
role.rbac.authorization.k8s.io/nginx-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/nginx-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/nginx-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/nginx-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/nginx-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-operator-proxy-rolebinding created
configmap/nginx-operator-manager-config created
service/nginx-operator-controller-manager-metrics-service created
deployment.apps/nginx-operator-controller-manager created
```

cat <<EOF | kubectl apply -f -
apiVersion: charts.jkosik/v1alpha1
kind: Nginx
metadata:
  name: nginx-sample
spec:
  replicaCount: 2
EOF

- `make undeploy` to uninstall Operator

#### (3) Bundling with OLM
The Operator Lifecycle Manager (OLM) is a set of cluster resources that manage the lifecycle of an Operator. The Operator SDK supports both creating manifests for OLM deployment, and testing your Operator on an OLM-enabled Kubernetes cluster.
Important: this guide assumes your project was scaffolded with operator-sdk init --project-version=3.

- `operator-sdk olm install` to install OLM
- `make bundle bundle-build bundle-push` to build OLM bundle and push to DockeHub => `docker push jkosik/nginx-operator-bundle:v0.0.1`
- `operator-sdk run bundle example.com/nginx-operator-bundle:v0.0.1` installs the bundle
- Operators DB: https://operatorhub.io/. Operators use OLM to install Operator into the cluster.