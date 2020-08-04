## helm cert-manager

[https://cert-manager.io/docs/installation/kubernetes/](https://cert-manager.io/docs/installation/kubernetes/)

Note: cert-manager should never be embedded as a sub-chart into other Helm charts. cert-manager manages non-namespaces resources in your cluster and should only be installed once.


### Install cert-manager

1. Add the repository:
``` helm repo add jetstack https://charts.jetstack.io
```

2. Update local Helm repo.
``` helm repo update
```

3. Create the namespace for cert-manager
``` kubectl create namespace cert-manager
```

4. Install the CRDs (CustomReasourceDefinitions) with kubectl. 
``` kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.16.0/cert-manager.crds.yaml
```
**Notes:**
* Install CRD seperatly from Helm makes it simplier to uninstall the CRD later.  See in Helm chart docs. "--set installCRDs=true"
* See install details for K8 v1.16 - requires Helm v3.2

**Outputs:**
```customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
```

5. Install cert-manager with Helm 
``` helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v0.16.0
```
**NOTES**
* version optional

6. Verify install
``` kubectl get pods --namespace cert-manager
```

7. Create an Issuer to test the webhook
```cat <<EOF > test-resources.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  dnsNames:
    - example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF
```

8. Create the resource

``` kubectl apply -f dev/test-resources.yaml
```

9. Test the resource.

``` kubectl delete -f dev/test-resources.yaml
```

## Uninstall cert-manager

[https://hub.helm.sh/charts/jetstack/cert-manager](https://hub.helm.sh/charts/jetstack/cert-manager)

1. Uninstall the Helm Chart by deleting the helm deployment.
``` helm delete cert-manager --namespace cert-manager
```

2. Delete the CustomResourceDefinition resources.
``` kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v0.16.0/cert-manager.crds.yaml
```
**Outputs:**
```customresourcedefinition.apiextensions.k8s.io "certificaterequests.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "certificates.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "challenges.acme.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "clusterissuers.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "issuers.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "orders.acme.cert-manager.io" deleted
```
