# 4KUBE-EXAM

PROJECT : Mathys GOUDAL - Simon LALANE

For this project, we had to set up a distributed application to track a fleet of delivery vehicles in real time.

To do this, we had to :
 1) Create a deployment for each container.

 2) Create an internal service for each deployment.

 3) Use a volume for the database. 

 4) Set up probes to monitor the pods.

***
## Useful commands

### Project launch

Command to generate Kubernetes configuration files from a Helm :
```bash
helm template . > manifest.yaml
```

Command to install the current directory's Chart Helm in your Kubernetes cluster, giving it the release name "kubeexam". After executing this command, the resources defined in the chart Helm will be deployed in your cluster:
```bash
helm install kubeexam .
```

### Pods

Command to display the list of pods created:
```bash
kubectl get pods
```

Command to display information about the selected pod:
```bash
kubectl describe pod [idPod]
```

### Deployments

Command to display the list of created deployments:
```bash
kubectl get deployments
```

Command to display info for the selected deployment:
```bash
kubectl describe deployment [idDeployment]
```

### Services

Command to display the list of services created:
```bash
kubectl get services
```


Command to display information about the selected service:
```bash
kubectl describe service [idService]
```


## Project File Structure

```
4KUBE-EXAM/
|   templates/
|   |-- deployments.yaml
|   |-- persistentVolumeClaim.yaml
|   |-- secret.yaml
|   |-- service.yaml
|-- Chart.yaml
|-- LICENSE
|-- manifest.yaml
|-- README.md
|-- values.yaml
```

4KUBE-EXAM/ : Project root folder
- templates/ : Folder containing YAML templates for Kubernetes resources.
- deployments.yaml: YAML file describing application deployments.
- persistentVolumeClaim.yaml: YAML file defining persistent volume claims.
- secret.yaml: YAML file containing a small amount of sensitive data.
- service.yaml: YAML file for defining services.
- Chart.yaml: Main Helm chart configuration file, specifying chart metadata.
- LICENSES: License file for project ownership rights.
- manifest.yaml: Main manifest file for the project.
- README.md: Project documentation, providing information on use, installation and other relevant details.
- values.yaml: Default values file for chart configuration.


***

## Chart

```yaml
apiVersion: v2
```
Specifies the version of the Helm chart format. Helm uses different API versions to ensure compatibility with different versions of Helm.

```yaml
name: .
```
Defines the name of the Helm chart. In this case, it appears to be the current directory (indicated by .). The name is a unique identifier for your chart.

```yaml
description: A Helm chart for Kubernetes
```
Provides a brief description of the Helm chart, explaining its purpose. This description is generally used to give users an idea of the chart's functionality.

```yaml
type: application
```
Specifies the type of Helm chart. In this case, it is marked as an application. The type gives an indication of whether the chart is an application, library or service.

```yaml
version: 0.1.0
```
Indicates the version of the Helm chart itself. It follows the semantic version format (Major.Minor.Fix), and this version number is used to track changes and updates to the chart.
```yaml
appVersion: "1.16.0"
```
Specifies the version of the application or service that the Helm chart deploys. This is distinct from the chart version and tracks the version of the application deployed by the chart.

In summary, the chart.yaml file provides essential metadata for your Helm chart, including its name, description, type, version and the version of the application it deploys. It helps Helm users and the Helm system to understand and manage the chart.


## Deployments

#### Overview
This code generates Kubernetes deployment definitions using the values provided in the configuration file.

#### Iteration on Deployments
The code uses a range loop to iterate over each deployment defined in the values file.

```yaml
{{- range $key, $value := .Values.deployments }}
---
```

#### Deployment definition
Each deployment is defined as a Kubernetes object of type "Deployment" (apps/v1).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ $key }}
  labels:
    app: {{ $key }}
spec:
  ...
```

#### Deployment configuration
The spec block contains the specific deployment configuration, including number of replicas, pod selector, containers, volumes and environment variables.

```yaml
spec:
  replicas: {{ $value.replicas | default $.Values.global.replicaCount }}
  selector:
    matchLabels:
      app: {{ $key }}
  template:
    metadata:
      labels:
        app: {{ $key }}
    spec:
      containers:
        ...
      volumes:
        ...
```

- Replicas : Defines the number of replicas for deployment.
- Pod selector: Specifies the pod selector for deployment.
- Containers: Defines containers to run in pods, including image, image pull policy, environment variables, and volume mount points.
- Volumes: Defines volumes to be mounted in pods, where applicable.
- Container names
- Container names are derived from the deployment name.

```yaml
- name: {{ $key }}
```


#### Container image
The code below dynamically constructs the container image URL according to the specified repository path, with conditional logic to handle absolute and relative references. The image retrieval policy is also defined to optimize performance by avoiding retrieving the image if it is already present

```yaml
{{- if hasPrefix "/" $value.image.repository }}
image: "{{ $.Values.global.repository }}{{ $value.image.repository }}:{{ $value.image.tag | default $.Values.global.image.tag }}"
{{- else }}
image: "{{ $value.image.repository }}:{{ $value.image.tag | default $.Values.global.image.tag }}"
{{- end}}
imagePullPolicy: "IfNotPresent"
```

#### Environment variablesEnvironment variables are defined according to deployment specifications and global variables.

```yaml
{{- if not (eq $value.environment "disable") }}
envFrom:
  - secretRef:
    name: env-secret
{{- end }}
```

#### Resource management
The configuration block below is used to define Kubernetes resources (CPU and memory) for deployments. It uses the values provided in the configuration file.

- Limits :    - CPU: The maximum amount of CPU a container can use.  - Memory: The maximum amount of memory a container can use.- Requests:  - CPU: The initial amount of CPU the container requests.  - Memory: The initial amount of memory requested by the container.The code uses conditions to check whether limit values and requests are defined in the value file, and uses them if necessary.

```yaml
 resources:
    limits:
      cpu: {{- if index $.Values.global.resources "limits" }}
        {{- if index $.Values.global.resources.limits "cpu" }} {{ $.Values.global.resources.limits.cpu }}{{ end }}
      {{- end }}
      memory: {{- if index $.Values.global.resources "limits" }}
        {{- if index $.Values.global.resources.limits "memory" }} {{ $.Values.global.resources.limits.memory }}{{ end }}
      {{- end }}
    requests:
      cpu: {{- if index $.Values.global.resources "requests" }}
        {{- if index $.Values.global.resources.requests "cpu" }} {{ $.Values.global.resources.requests.cpu }}{{ end }}
      {{- end }}
      memory: {{- if index $.Values.global.resources "requests" }}
        {{- if index $.Values.global.resources.requests "memory" }} {{ $.Values.global.resources.requests.memory }}{{ end }}
      {{- end }}
```

#### Probes

The code below is a set of templates for defining liveness probes and readiness probes in YAML Kubernetes configuration files. Here's an explanation of the structure and use of these templates

Condition checking for probes

```yaml
{{- if hasKey $value "probes" }}
```

Vitality Probe configuration
```yaml
{{- if hasKey $value.probes "livenessProbe" }}
livenessProbe:
  {{- if hasKey $value.probes.livenessProbe "command" }}
  exec:
    command:
      {{- range $value.probes.livenessProbe.command }}
      - {{ . }}
      {{- end }}
  {{- end }}
  {{- if hasKey $value.probes.livenessProbe "httpGet" }}
  httpGet:
    path: {{ $value.probes.livenessProbe.httpGet.path | default "/" }}
    port: {{ $value.probes.livenessProbe.httpGet.port | default 80 }}
    scheme: {{ $value.probes.livenessProbe.httpGet.scheme | default "HTTP" }}
    host: {{ $value.probes.livenessProbe.httpGet.host | default "localhost" }}
  {{- end }}
  {{- if hasKey $value.probes.livenessProbe "tcpSocket" }}
  tcpSocket:
    port: {{ $value.probes.livenessProbe.tcpSocket.port | default 80 }}
  {{- end }}            
  initialDelaySeconds: {{ $value.probes.livenessProbe.initialDelaySeconds | default 1 }}
  periodSeconds: {{ $value.probes.livenessProbe.periodSeconds | default 10 }}
  timeoutSeconds: {{ $value.probes.livenessProbe.timeoutSeconds | default 5 }}
  successThreshold: {{ $value.probes.livenessProbe.timeoutSeconds | default 1 }}
  failureThreshold: {{ $value.probes.livenessProbe.timeoutSeconds | default 2 }}
{{- end }}
```

Availability probe configuration

```yaml
{{- if hasKey $value.probes "readinessProbe" }}
readinessProbe:
  {{- if hasKey $value.probes.readinessProbe "command" }}
  exec:
    command:
      {{- range $value.probes.readinessProbe.command }}
      - {{ . }}
      {{- end }}
  {{- end }}
  {{- if hasKey $value.probes.readinessProbe "httpGet" }}
  httpGet:
    path: {{ $value.probes.readinessProbe.httpGet.path | default "/" }}
    port: {{ $value.probes.readinessProbe.httpGet.port | default 80 }}
    scheme: {{ $value.probes.readinessProbe.httpGet.scheme | default "HTTP" }}
    host: {{ $value.probes.readinessProbe.httpGet.host | default "localhost" }}
  {{- end }}
  {{- if hasKey $value.probes.readinessProbe "tcpSocket" }}
  tcpSocket:
    port: {{ $value.probes.readinessProbe.tcpSocket.port | default 80 }}
  {{- end }}
  initialDelaySeconds: {{ $value.probes.readinessProbe.initialDelaySeconds | default 1 }}
  periodSeconds: {{ $value.probes.readinessProbe.periodSeconds | default 10 }}
  timeoutSeconds: {{ $value.probes.readinessProbe.timeoutSeconds | default 5 }}
  successThreshold: {{ $value.probes.readinessProbe.timeoutSeconds | default 1 }}
  failureThreshold: {{ $value.probes.readinessProbe.timeoutSeconds | default 2 }}
{{- end }}
{{- end }}
```

**Use** :
1. Condition check for probes ```{{- if hasKey $value "probes" }}``` :
- This check ensures that subsequent sections are only processed if the key "probes" is present in $value.
2. Vitality Probe configuration ```{- if hasKey $value.probes "livenessProbe" }} ... {{- end }}``` :
- Configures the vitality probe according to the presence of the "livenessProbe" key in the "probes" section of $value.
3. Availability Probe configuration ```{{- if hasKey $value.probes "readinessProbe" }} ... {{- end }}``` :
- ²Configures the availability probe according to the presence of the "readinessProbe" key in the "probes" section of $value.

#### Volume mount points
Volume mount points are defined if specified in the deployment.

```yaml
volumeMounts:
  - mountPath: {{ $value.volume.mountPath }}
    name: {{ $key }}
```

#### End of loop
The range loop ends at the end of the code.

```yaml
{{- end }}
```

This configuration generates Kubernetes deployments based on the specifications provided in the values file. Each deployment has a specific name and configuration, with the option of defining replicas, containers, volumes and custom environment variables.

## Services

#### Overview
This code generates Kubernetes service definitions using the values provided in the configuration file.

#### Iterating on Services
The code uses a range loop to iterate over each service defined in the values file.

```yaml
{{- range $key, $value := .Values.services }}
---
```

#### Service definition
Each service is defined as a Kubernetes object of type "Service" (v1).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{$.Values.global.name }}-{{ $key }}
  labels:
    app: {{ $key }}
spec:
  ...
```

#### Service configuration
The spec block contains service-specific configuration, including service type, cluster IP address, ports, and associated pod selection.
``` yaml
spec:
  {{- if $value.type }}
  type: {{ $value.type }}
  {{- end}}
  {{- if $value.clusterIp }}
  clusterIP: {{ $value.clusterIp }}
  {{- end}}
  {{- with $value.ports }}
  ports:
    {{- range $portKey, $portValue := $value.ports }}
    ...
    {{- end}}
  {{- end}}
  selector:
    app: {{ $key }}
```

- Service type (type): Defines the type of service (LoadBalancer, ClusterIP, NodePort, etc.).
- Cluster IP address (clusterIP): Specifies the IP address of the service in the cluster.
- Ports: Defines the service's ports, with the option of specifying an ID and NodePort for each port.
- Pod selector: Selects the pods associated with the service using a specific label.

#### Port names
Port names are derived from the service name and, where applicable, the port ID.
```yaml
- name: {{ $key -}}-{{ $portValue.id -}}-service
```

#### End of loop
The range loop ends at the end of the code.
```yaml
{{- end }}`
```
This configuration generates Kubernetes services based on the specifications provided in the values file. Each service is named according to the global service name and the specific service name defined in the values file.

## Persistent Volume Claim

Overview
This code generates Kubernetes persistent volume claim (PVC) definitions using the values provided in the configuration file.

Iterating on Persistent Volumes
The code uses a range loop to iterate over each persistent volume defined in the values file.

```yaml
{{- range $key, $value := .Values.volumes }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ $key }}
spec:
  ...
{{- end }}
```

#### Persistent Volume Claim definition
Each persistent volume claim is defined as a Kubernetes object of type "PersistentVolumeClaim" (v1).

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ $key }}
spec:
  ...
```

#### Persistent Volume Claim configuration
The `spec` block contains the specific configuration of the persistent volume claim, including access modes and requested resources.

```yaml
spec:
  accessModes: 
    ...
  resources:
    requests:
      ...
```

**AccessModes:** Defines volume access modes, using the value specified in the values file or the global default.

```yaml
accessModes: 
  {{- if $value.accessModes }}
  - {{ $value.accessModes }}
  {{- else }}
  - {{ $.Values.global.volume.accessMode }}
  {{- end}}
```

**Resources Requested (resources.requests.storage) :** Defines the amount of storage requested for the volume, using the value specified in the values file or the global default value.

```yaml
resources:
  requests:
    {{- if $value.resources.requests.storage }}
    storage: {{ $value.resources.requests.storage }}
    {{- else }}
    storage: {{ $.Values.global.volume.storage }}
    {{- end}}
```

#### End of loop
The `range` loop ends at the end of the code.

```yaml
{{- end }}
```

This configuration generates Kubernetes persistent volume claims based on the specifications provided in the values file. Each persistent volume claim has a specific name and detailed configuration, with the ability to define custom access modes and storage resources.

## Secret

### Secret Kubernetes object
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: env-secret
type: Opaque
data:
  SPRING_PROFILES_ACTIVE: cHJvZHVjdGlvbi1taWNyb3NlcnZpY2U=
```

This Secret object is used to store sensitive data, such as credentials or API keys, securely in a Kubernetes cluster.

### Secret Object metadata

```yaml
metadata:
  name: env-secret
```

Metadata provides descriptive information about the Secret object.
name: The name assigned to this Secret object, in this example, "env-secret".
Secret type

```yaml
type: Opaque
```

The "type" property defines the type of Secret. In this example, the "Opaque" type is used, which means that the Secret can contain arbitrary data and is not structured.

### Secret data
```yaml
data:
  SPRING_PROFILES_ACTIVE: cHJvZHVjdGlvbi1taWNyb3NlcnZpY2U=
```

The "data" section contains the key-value pairs of sensitive data stored in the Secret.
 - **SPRING_PROFILES_ACTIVE**: This is a specific key in the Secret, used to store information about active Spring profiles.
