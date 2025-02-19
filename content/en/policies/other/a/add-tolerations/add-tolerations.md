---
title: "Add Tolerations"
category: Other
version: 1.6.0
subject: Pod
policyType: "mutate"
description: >
    Pod tolerations are used to schedule on Nodes which have a matching taint. This policy adds the toleration `org.com/role=service:NoSchedule` if existing tolerations do not contain the key `org.com/role`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/a/add-tolerations/add-tolerations.yaml" target="-blank">/other/a/add-tolerations/add-tolerations.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-tolerations
  annotations:
    policies.kyverno.io/title: Add Tolerations
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.7.1
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >- 
      Pod tolerations are used to schedule on Nodes which have
      a matching taint. This policy adds the toleration `org.com/role=service:NoSchedule`
      if existing tolerations do not contain the key `org.com/role`.
spec:
  rules:
  - name: service-toleration
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      any:
      - key: "org.com/role"
        operator: AnyNotIn
        value: "{{ request.object.spec.tolerations[].key || `[]` }}"
    mutate:
      patchesJson6902: |-
        - op: add
          path: "/spec/tolerations/-"
          value:
            key: org.com/role
            operator: Equal
            value: service
            effect: NoSchedule

```
