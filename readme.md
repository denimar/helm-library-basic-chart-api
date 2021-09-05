## Simple guide.

This chart is intentionally kept simple to support a web api.
optionally you can specify a `healthCheckPort` which then looks for liveness and readiness probes configured at `/health/live` and `/health/ready`. Rest of the values are explanatory. 

### Configuration per environment
- Look at `env` folder.
- create a yaml file such as `config.{env}.yaml`.

### secrets
- secrets are injected as config map
- store them in `env` folder with name such as `secrets.{env}.yaml`
- you will need to use helm secrets to edit encrypted secrets.

### installing the charts
```
helm secrets install bg helm/ --set env=dev --values helm/env/secrets.dev.yaml
```

### Using this chart - help.
1. create a basic helm chart: `helm create <your chart>`
2. Delete all files under "templates".
3. Create a single file with the following contents. This will import the needed pieces
```
{{ include "basicapi.configmap" .}}
---
{{ include "basicapi.deployment" .}}
---
```
4. Go to Chart.yaml and add the following åt the end. 
```
dependencies:
  - name : basicapi
    version: <version>
    repository: <repo>
```
Then update using `helm dependency update <chart path>`.
5. Create a values.yaml, env/config.dev.yaml, env/secrets.dev.yaml
```
#values.yaml
env: dev
labels:
  system: oxygen
  jobKind: background
replicas: 1
container:
  image: krishna/background
  tag: latest
#healthCheckPort: 3000
httpPort: 80
resources: 
  limits:
    cpu: 300m
    memory: 256Mi
  requests:
    cpu: 300m
    memory: 256Mi
migrations:
  image: #if not set, uses the container image itself
  command:
  args: 
ingressHostName: dev-hydrogen-api.shopolive.team
configMapFile: # when specified, this will replace config.<env>.yaml. Note that config is still looked for within the env directory.
secretsFile: # when specified, this will replace secrets.<env>.yaml. Note that config is still looked for within the env directory.
```
6. You may want to configure `.sops.yaml` for handling secrets. And encrypt using `helm secrets enc helm-chart/env/secrets.dev.yaml`
7. Additional commands that are helpful.
```
helm template chartname/ --debug
helm secrets install bg chartname/ --set env=dev --values deploy/env/secrets.dev.yaml
heml uninstall bg
```
8. Note that this this will automatically enable TLS using LetsEncrypt.

## More changes.
1. Added support for specifying annotations on deployments. For example:
```
#values.yaml
annotations:
  dapr.io/enabled: "true"
  dapr.io/app-id: "hello"
  dapr.io/app-port: "3000"
  dapr.io/log-as-json: "true"
```
2. Added support to specify additional ports when creating a service using `morePorts`. Note that the default port will be named as default unless specified in `httpPortName`. For example:
```
httpPort: 3500
morePorts:
  - port: 9090
    name: metrics
```
3. Updates to the ingress configurations.
  - You can override the ingress class through values by setting `ingressClass` value.
  - You can configure additional ingresses using the `moreHttpIngresses` array. Each element in the array shall have `ingressHostName`, `ingressPath`, `ingressServiceName`, `ingressServicePort`. For example:
  ```
  moreHttpIngresses:
    - ingressHostName: rhone.shopolive.com
      ingressPath: / #defaults to / as well.
      ingressServiceName: hydrogen-deliveries-service
      ingressServicePort: 8080
    - ingressHostName: nike.shopolive.com
      ingressPath: / #defaults to / as well.
      ingressServiceName: hydrogen-deliveries-service
      ingressServicePort: 8080
  ```
  - Note that this particular functionality has not been tested.