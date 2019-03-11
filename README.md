# Build (already done)
<pre>
# uses the docker file from build dir
operator-sdk build quay.io/${QUAY_ID}/gogs-operator:v0.0.1
docker push quay.io/${QUAY_ID}/gogs-operator:v0.0.1
</pre>

# Deploy
<pre>
# Create the custome resource definition
oc create -f ./deploy/crds/gpte_v1alpha1_gogs_crd.yaml

# Create rules
echo '---
apiVersion: authorization.openshift.io/v1
kind: ClusterRole
metadata:
  labels:
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
  name: gogs-admin-rules
rules:
- apiGroups:
  - gpte.opentlc.com
  resources:
  - gogs
  - gogs/status
  verbs:
  - create
  - update
  - delete
  - get
  - list
  - watch
  - patch' | oc create -f -


oc create -f ./deploy/service_account.yaml
oc create -f ./deploy/role.yaml
oc create -f ./deploy/role_binding.yaml

# Deploy operator
oc create -f ./deploy/operator.yaml

# Deploy gogs cluster
oc create -f $HOME/gogs-operator/gogs-server.yaml

# Check 
oc get gogs
</pre>
