This policy generates a definition yaml with the correct timezone from a pod
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: enforce-timezone-bern
spec:
  rules:
    - name: enforce-timezone
      match:
        resources:
          kinds:
            - Pod
      generate:
        kind: Pod
        name: enforce-timezone-pod
        namespace: "{{request.object.metadata.namespace}}"
        data:
          apiVersion: v1
          kind: Pod
          metadata:
            name: enforce-timezone-pod
          spec:
            containers:
              - name: "{{request.object.spec.containers[0].name}}"
                image: "{{request.object.spec.containers[0].image}}"
                env:
                  - name: TZ
                    value: "Europe/Zurich"
```