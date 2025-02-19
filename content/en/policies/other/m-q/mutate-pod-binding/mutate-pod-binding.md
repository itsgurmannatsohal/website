---
title: "Mutate Pod Binding"
category: Other
version: 1.10.0
subject: Pod
policyType: "mutate"
description: >
    Containers running in Pods may sometimes need access to node-specific information on which the Pod has been scheduled. Scheduling decisions are made by kube-scheduler after the Pod has been persisted and so only at that time may the Node to which the Pod is bound can be fetched. The Kubernetes API allows specifically the projection of annotations from these Binding resources to the Pods which are their subject. This policy watches for then mutates the /binding subresource of a Pod to add an annotation named `foo` the value of which comes from the bound Node's label also called `foo`. Use of this policy may require removal of the Binding resourceFilter in Kyverno's ConfigMap.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/m-q/mutate-pod-binding/mutate-pod-binding.yaml" target="-blank">/other/m-q/mutate-pod-binding/mutate-pod-binding.yaml</a>

```yaml
apiVersion: kyverno.io/v2beta1
kind: ClusterPolicy
metadata:
  name: mutate-pod-binding
  annotations:
    pod-policies.kyverno.io/autogen-controllers: none
    policies.kyverno.io/title: Mutate Pod Binding
    policies.kyverno.io/category: Other
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.10.0
    policies.kyverno.io/minversion: 1.10.0
    kyverno.io/kubernetes-version: "1.26"
    policies.kyverno.io/description: >-
      Containers running in Pods may sometimes need access to node-specific information
      on which the Pod has been scheduled. Scheduling decisions are made by kube-scheduler after
      the Pod has been persisted and so only at that time may the Node to which the Pod is bound
      can be fetched. The Kubernetes API allows specifically the projection of annotations from these
      Binding resources to the Pods which are their subject. This policy watches for then mutates
      the /binding subresource of a Pod to add an annotation named `foo` the value of which comes
      from the bound Node's label also called `foo`. Use of this policy may require removal of the
      Binding resourceFilter in Kyverno's ConfigMap.
spec:
  background: false
  rules:
    - name: project-foo
      match:
        any:
        - resources:
            kinds:
            - Pod/binding
      context:
      - name: node
        variable:
          jmesPath: request.object.target.name
          default: ''
      - name: foolabel
        apiCall:
          urlPath: "/api/v1/nodes/{{node}}"
          jmesPath: metadata.labels.foo || 'empty'
      mutate:
        patchStrategicMerge:
          metadata:
            annotations:
              foo: "{{ foolabel }}"
```
