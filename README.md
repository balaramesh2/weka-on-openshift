# WEKA On OpenShift
This README explains the steps to be taken to deploy WEKA on OpenShift 4.20 and above.

# PREREQUISITES

- A working OpenShift cluster. A non-HCP cluster is required.

  *UPDATE: Hosted Control Plane clusters do not work, on account of certain required CRDs not being exposed (such as `MachineConfig`)*

# STEPS

1. Deploy WEKA Operator v1.9.0 or newer:

```
helm upgrade --create-namespace \
    --install weka-operator oci://quay.io/weka.io/helm/weka-operator \
    --namespace weka-operator-system \
    --version v1.9.0 \
    --set csi.installationEnabled=true
```
Confirm the operator is up and running:
```
oc get pods -n weka-operator-system
```
You should see 1 controller pod + `n` node pods (where `n` equals number of OpenShift nodes)

2. Create wekaPolicy objects:

```
oc create -f wekapolicy.yaml
```
The wekaPolicy objects sign available hard drives and use them to build a wekaCluster.

Confirm the wekaPolicy worked as expected:

```
oc describe node | grep weka.io/
```
Each node should have the following annotations added to them after a successful wekaPolicy execution:
- `weka.io/weka-drives`
- `weka.io/sign-drives-hash`
- `weka.io/discovery.json`

You can also check the output of `oc get wekaPolicy --all-namespaces`

3. Create a wekaCluster object:
```
oc create -f wekacluster.yaml
```
Observe the progression of cluster deployment with:
```
watch oc get pods,wekacluster -n weka-operator-system
```
wekaCluster should progress through the following stages: `Init`, `ReadyForIO`, `Ready`
