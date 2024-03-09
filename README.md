# Kustomize Namespace Bug

Kustomize doesn't pass down the namespace when defined on the top level. This causes inconsitent YAML files to be built.

You can easily test it yourself by running `kustomize build . --enable-helm` in the root of this repository.

## kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

helmGlobals:
  chartHome: ./charts

namespace: the-actual-namespace   # <- defines namespace for all resources

helmCharts:
- name: service
  releaseName: service
  valuesFile: values.yaml
```

## Result

```yaml
$ kustomize build . --enable-helm
apiVersion: v1
kind: Service
metadata:
  annotations:
    this-service-is-deployed-in-namespace: default # <- {{ .Release.Namespace }} still points to "default"
  name: the-bug
  namespace: the-actual-namespace                  # <- Namespace correctly set
```
