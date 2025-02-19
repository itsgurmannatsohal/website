---
title: "Inject Env Var from Image Label"
category: Other
version: 1.7.0
subject: Pod
policyType: "mutate"
description: >
    Container images which use metadata such as the LABEL directive in a Dockerfile do not surface this information to apps running within. In some cases, running the image as a container may need access to this information. This policy injects the value of a label set in a Dockerfile named `maintainer` as an environment variable to the corresponding container in the Pod.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/e-l/inject-env-var-from-image-label/inject-env-var-from-image-label.yaml" target="-blank">/other/e-l/inject-env-var-from-image-label/inject-env-var-from-image-label.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: inject-env-var-from-image-label
  annotations:
    policies.kyverno.io/title: Inject Env Var from Image Label
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    pod-policies.kyverno.io/autogen-controllers: none
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.7.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Container images which use metadata such as the LABEL directive in a Dockerfile
      do not surface this information to apps running within. In some cases, running the image
      as a container may need access to this information. This policy injects the value of a label
      set in a Dockerfile named `maintainer` as an environment variable to the corresponding container
      in the Pod.
spec:
  rules:
  - name: add-env-maintainer
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    mutate:
      foreach:
      - list: "request.object.spec.containers"
        context: 
        - name: maintainer
          imageRegistry: 
            reference: "{{ element.image }}"
            jmesPath: "configData.config.Labels.maintainer || ''"
        preconditions:
          all:
          - key: "{{maintainer}}"
            operator: NotEquals
            value: ""
        patchesJson6902: |-
          - op: add
            path: "/spec/containers/{{elementIndex}}/env/-"
            value:
              name: MAINTAINER
              value: "{{maintainer}}"
```
