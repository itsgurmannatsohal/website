---
title: "Ingress Host Match TLS"
category: Other
version: 1.6.0
subject: Ingress
policyType: "validate"
description: >
    Ingress resources which name a host name that is not present in the TLS section can produce ingress routing failures as a TLS certificate may not correspond to the destination host. This policy ensures that the host name in an Ingress rule is also found in the list of TLS hosts.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/e-l/ingress-host-match-tls/ingress-host-match-tls.yaml" target="-blank">/other/e-l/ingress-host-match-tls/ingress-host-match-tls.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: ingress-host-match-tls
  annotations:
    policies.kyverno.io/title: Ingress Host Match TLS
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.20, 1.21"
    policies.kyverno.io/subject: Ingress
    policies.kyverno.io/description: >-
      Ingress resources which name a host name that is not present
      in the TLS section can produce ingress routing failures as a TLS
      certificate may not correspond to the destination host. This policy
      ensures that the host name in an Ingress rule is also found
      in the list of TLS hosts.
spec:
  background: false
  validationFailureAction: audit
  rules:
  - name: host-match-tls
    match:
      any:
      - resources:
          kinds:
          - Ingress
    preconditions:
      all:
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: "The host(s) in spec.rules[].host must match those in spec.tls[].hosts[]."
      deny:
        conditions:
          all:
          - key: "{{ (request.object.spec.rules[].host || `[]`) | sort(@) }}"
            operator: AnyNotIn
            value: "{{ (request.object.spec.tls[].hosts[] || `[]`) | sort(@) }}"

```
