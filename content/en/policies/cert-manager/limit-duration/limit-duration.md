---
title: "Certificate max duration 100 days"
category: Cert-Manager
version: 1.3.6
subject: Certificate
policyType: "validate"
description: >
    k8s managed non-letsencrypt certificates have to be renewed in every 100day
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//cert-manager/limit-duration/limit-duration.yaml" target="-blank">/cert-manager/limit-duration/limit-duration.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: cert-manager-limit-duration
  annotations:
    policies.kyverno.io/title: Certificate max duration 100 days
    policies.kyverno.io/category: Cert-Manager
    policies.kyverno.io/severity: medium
    policies.kyverno.io/minversion: 1.3.6
    policies.kyverno.io/subject: Certificate
    policies.kyverno.io/description: k8s managed non-letsencrypt certificates have to be renewed in every 100day
spec:
  validationFailureAction: enforce
  background: false
  rules:
  - name: certificate-duration-max-100days 
    match:
      resources:
        kinds:
        - Certificate
    preconditions:
      all:
      - key: "{{ contains(request.object.spec.issuerRef.name, 'letsencrypt') }}"
        operator: Equals
        value: False
      - key: "{{ request.object.spec.duration }}"
        operator: NotEquals
        value: ""
    validate:
      message: "certificate duration must be < than 2400h (100 days)"
      deny:
        conditions:
        - key: "{{ max( [ to_number(regex_replace_all('h.*',request.object.spec.duration,'')), to_number('2400') ] ) }}"
          operator: NotEquals
          value: 2400
```
