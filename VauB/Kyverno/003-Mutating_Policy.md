This policy mutates the existing Environment variable "TZ" to "Europe/Zurich"
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: inject-timezone
spec:
  background: true
  rules:
  - name: inject-timezone
    match:
      any:
      - resources:
          kinds:
          - Pod
    mutate:
      patchStrategicMerge:
        spec:
          containers:
          - (name): "*"
            env:
            - name: TZ
              value: Europe/Zurich
```