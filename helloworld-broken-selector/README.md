# Helloworld-Broken-Selector

A helloworld chart with a broken selector. The key, `app.kubernetes.io/managed-by`, is fixed, but the value varies with the Helm version. After the chart is migrated from Helm v2 to v3, it cannot be upgraded, because the selector value changes, but a selector is an immutable field.

This issue has been seen in the wild. For example, https://github.com/jetstack/cert-manager/issues/2451.

In general, if the field is immutable, do not set its value dynamically.

## Reproduce Upgrade Error

Make sure you have these dependencies installed:

- kind
- helm2
- helm3

Then, run this script:

```shell
# Prepare cluster
kind create cluster

# Prepare helm2
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
helm2 init --service-account tiller --wait

# Prepare helm3
helm3 plugin install https://github.com/helm/helm-2to3.git

# Install chart
helm2 install --name broken --namespace default ./0.0.1

# Migrate chart
helm3 2to3 convert broken

# Upgrade chart
helm3 upgrade broken ./0.0.2
```

The upgrade should fail with the error:

```shell
Error: UPGRADE FAILED: cannot patch "broken-helloworld-broken-selector" with kind Deployment: Deployment.apps "broken-helloworld-broken-selector" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app.kubernetes.io/instance":"broken", "app.kubernetes.io/managed-by":"Helm", "app.kubernetes.io/name":"helloworld-broken-selector"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
```
