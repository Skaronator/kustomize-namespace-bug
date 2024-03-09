# Kustomize Namespace Bug

Bug report: [kubernetes-sigs/kustomize#5566](https://github.com/kubernetes-sigs/kustomize/issues/5566)

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

## Helm Chart Template

```yaml
apiVersion: v1
kind: Service
metadata:
  name: the-bug
  namespace: {{ .Release.Namespace }}
  annotations:
    this-service-is-deployed-in-namespace: {{ .Release.Namespace }}
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
  namespace: the-actual-namespace                  # <- Namespace overwritten by Kustomization to the correct value
```
