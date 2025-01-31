---
title: "Remove ServiceAccount Token"
category: Other
version: 1.10.0
subject: Pod,ServiceAccount,Volume
policyType: "mutate"
description: >
    Pods running with a ServiceAccount are presented with a volume, containing the token, and volume mounts for all containers in the Pod. Applications that do not need to communicate with the Kubernetes API do not need a ServiceAccount and therefore limiting which Pods have access rights is important. Rather than, or in addition to, requiring that certain Pods disable mounting of a ServiceAccount, it is possible to silently remove this token if it has been presented. This policy ensures that Pods which do not have the label `corp.org/can-use-serviceaccount` and are consuming a ServiceAccount have that stripped away. It should be customized to restrict the scope of its operation as it will not distinguish between an explicitly-defined ServiceAccount or one provided by default.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/rec-req/remove-serviceaccount-token/remove-serviceaccount-token.yaml" target="-blank">/other/rec-req/remove-serviceaccount-token/remove-serviceaccount-token.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: remove-serviceaccount-token
  annotations:
    policies.kyverno.io/title: Remove ServiceAccount Token
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod,ServiceAccount,Volume
    kyverno.io/kyverno-version: 1.10.0
    policies.kyverno.io/minversion: 1.10.0
    kyverno.io/kubernetes-version: "1.25"
    policies.kyverno.io/description: >-
      Pods running with a ServiceAccount are presented with a volume, containing
      the token, and volume mounts for all containers in the Pod. Applications that
      do not need to communicate with the Kubernetes API do not need a ServiceAccount and
      therefore limiting which Pods have access rights is important. Rather than, or in addition to,
      requiring that certain Pods disable mounting of a ServiceAccount, it is possible to silently
      remove this token if it has been presented. This policy ensures that Pods which do not
      have the label `corp.org/can-use-serviceaccount` and are consuming a ServiceAccount
      have that stripped away. It should be customized to restrict the scope of its operation as it
      will not distinguish between an explicitly-defined ServiceAccount or one provided by default.
spec:
  background: false
  rules:
    - name: remove-vol-volmount
      match:
        any:
        - resources:
            kinds:
            - Pod
            selector:
              matchExpressions:
              - key: corp.org/can-use-serviceaccount
                operator: DoesNotExist
      context:
      - name: tokenvolname
        variable:
          jmesPath: request.object.spec.volumes[?projected].name[?starts_with(@,'kube-api-access-')] | [0] || ''
          default: ''
      preconditions:
        all:
        - key: "{{ tokenvolname }}"
          operator: Equals
          value: "?*"
      mutate:
        foreach:
          - list: request.object.spec.volumes[]
            order: Descending
            preconditions:
              all:
              - key: projected
                operator: AnyIn
                value: "{{ element.keys(@) }}"
              - key: "{{ element.name }}"
                operator: Equals
                value: "kube-api-access-*"
            patchesJson6902: |-
              - path: /spec/volumes/{{elementIndex}}
                op: remove
          - list: request.object.spec.containers[]
            foreach:
            - list: element.volumeMounts
              order: Descending
              preconditions:
                all:
                - key: "{{element.name}}"
                  operator: AnyIn
                  value: "{{ tokenvolname }}"
              patchesJson6902: |-
                - path: /spec/containers/{{elementIndex0}}/volumeMounts/{{elementIndex1}}
                  op: remove

```
