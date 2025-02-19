---
title: "Disallow SELinux in CEL expressions"
category: Pod Security Standards (Baseline) in CEL
version: 
subject: Pod
policyType: "validate"
description: >
    SELinux options can be used to escalate privileges and should not be allowed. This policy ensures that the `seLinuxOptions` field is undefined.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//pod-security-cel/baseline/disallow-selinux/disallow-selinux.yaml" target="-blank">/pod-security-cel/baseline/disallow-selinux/disallow-selinux.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-selinux
  annotations:
    policies.kyverno.io/title: Disallow SELinux in CEL expressions
    policies.kyverno.io/category: Pod Security Standards (Baseline) in CEL
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      SELinux options can be used to escalate privileges and should not be allowed. This policy
      ensures that the `seLinuxOptions` field is undefined.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: selinux-type
      match:
        any:
        - resources:
            kinds:
              - Pod
      validate:
        cel:
          expressions:
            - expression: >- 
                !has(object.spec.securityContext) ||
                !has(object.spec.securityContext.seLinuxOptions) ||
                !has(object.spec.securityContext.seLinuxOptions.type) ||
                object.spec.securityContext.seLinuxOptions.type == 'container_t' ||
                object.spec.securityContext.seLinuxOptions.type == 'container_init_t' ||
                object.spec.securityContext.seLinuxOptions.type == 'container_kvm_t'
              message: >-
                Setting the SELinux type is restricted. The field spec.securityContext.seLinuxOptions.type 
                must either be unset or set to one of the allowed values (container_t, container_init_t, or container_kvm_t).

            - expression: >-
                object.spec.containers.all(container, !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                !has(container.securityContext.seLinuxOptions.type) ||
                container.securityContext.seLinuxOptions.type == 'container_t' ||
                container.securityContext.seLinuxOptions.type == 'container_init_t' ||
                container.securityContext.seLinuxOptions.type == 'container_kvm_t')
              message: >-
                Setting the SELinux type is restricted. The field spec.containers[*].securityContext.seLinuxOptions.type 
                must either be unset or set to one of the allowed values (container_t, container_init_t, or container_kvm_t).

            - expression: >- 
                !has(object.spec.initContainers) ||
                object.spec.initContainers.all(container, !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                !has(container.securityContext.seLinuxOptions.type) ||
                container.securityContext.seLinuxOptions.type == 'container_t' ||
                container.securityContext.seLinuxOptions.type == 'container_init_t' ||
                container.securityContext.seLinuxOptions.type == 'container_kvm_t')
              message: >-
                Setting the SELinux type is restricted. The field spec.initContainers[*].securityContext.seLinuxOptions.type 
                must either be unset or set to one of the allowed values (container_t, container_init_t, or container_kvm_t).

            - expression: >- 
                !has(object.spec.ephemeralContainers) ||
                object.spec.ephemeralContainers.all(container, !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                !has(container.securityContext.seLinuxOptions.type) ||
                container.securityContext.seLinuxOptions.type == 'container_t' ||
                container.securityContext.seLinuxOptions.type == 'container_init_t' ||
                container.securityContext.seLinuxOptions.type == 'container_kvm_t')
              message: >-
                Setting the SELinux type is restricted. The field spec.ephemeralContainers[*].securityContext.seLinuxOptions.type 
                must either be unset or set to one of the allowed values (container_t, container_init_t, or container_kvm_t).
    - name: selinux-user-role
      match:
        any:
        - resources:
            kinds:
              - Pod
      validate:
        cel:
          expressions:
            - expression: >- 
                !has(object.spec.securityContext) ||
                !has(object.spec.securityContext.seLinuxOptions) ||
                (!has(object.spec.securityContext.seLinuxOptions.user) && !has(object.spec.securityContext.seLinuxOptions.role))
              message: >-
                Setting the SELinux user or role is forbidden. The fields
                spec.securityContext.seLinuxOptions.user and spec.securityContext.seLinuxOptions.role must be unset.

            - expression: >- 
                object.spec.containers.all(container, !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                (!has(container.securityContext.seLinuxOptions.user) && !has(container.securityContext.seLinuxOptions.role)))
              message: >-
                Setting the SELinux user or role is forbidden. The fields
                spec.containers[*].securityContext.seLinuxOptions.user and spec.containers[*].securityContext.seLinuxOptions.role must be unset.

            - expression: >- 
                !has(object.spec.initContainers) ||
                object.spec.initContainers.all(container, !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                (!has(container.securityContext.seLinuxOptions.user) && !has(container.securityContext.seLinuxOptions.role)))
              message: >-
                Setting the SELinux user or role is forbidden. The fields
                spec.initContainers[*].securityContext.seLinuxOptions.user and spec.initContainers[*].securityContext.seLinuxOptions.role must be unset.

            - expression: >- 
                !has(object.spec.ephemeralContainers) ||
                object.spec.ephemeralContainers.all(container, !has(container.securityContext) ||
                !has(container.securityContext.seLinuxOptions) ||
                (!has(container.securityContext.seLinuxOptions.user) && !has(container.securityContext.seLinuxOptions.role)))
              message: >-
                Setting the SELinux user or role is forbidden. The fields
                spec.ephemeralContainers[*].securityContext.seLinuxOptions.user and spec.ephemeralContainers[*].securityContext.seLinuxOptions.role must be unset.

```
