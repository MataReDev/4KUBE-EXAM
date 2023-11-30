# 4KUBE-EXAM

Pour ce projet nous devions mettre en place application distribuée permettant de suivre en temps réel une flotte de véhicules effectuant des livraisons.

Pour cela nous devions :
 1) Créer un déploiement pour chaque conteneur.

 2) Créer un service interne pour chaque déploiement.

 3) Utiliser un volume pour la base de données.

 4) Mettre en place des sondes pour contrôler les pods

***
## Les Commandes Utiles

### Lancement du projet

Commande pour générer le fichier manifest.yaml qui sert à lancer les pods :
```bash
helm template . > manifest.yaml
```

Commande pour installer le fichier manifest et créer les pods :
```bash
helm install kubeexam .
```

### Les pods

Commande pour afficher la liste des pods créés :
```bash
kubectl get pods
```

Commande pour afficher les infos du pod sélectionné :
```bash
kubectl describe pod [idPod]
```

### Les deployments

Commande pour afficher la liste des deployments créés :
```bash
kubectl get deployments
```


Commande pour afficher les infos du deployments sélectionné :
```bash
kubectl describe deployment [idDeployment]
```

### Les services

Commande pour afficher la liste des services créés :
```bash
kubectl get services
```


Commande pour afficher les infos du services sélectionné :
```bash
kubectl describe service [idService]
```


## La Structure des Fichiers du Projet

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

4KUBE-EXAM/ : Dossier racine du projet
- templates/ : Dossier contenant les modèles YAML pour les ressources Kubernetes.
- deployments.yaml: Fichier YAML décrivant les déploiements des applications.
- persistentVolumeClaim.yaml: Fichier YAML définissant les revendications de volume persistant.
- secret.yaml: Fichier YAML contenant une petite quantité de données sensibles.
- service.yaml: Fichier YAML pour la définition des services.
- Chart.yaml: Fichier de configuration principal du chart Helm, spécifiant les métadonnées du chart.
- LICENCES : Fichier de licence concernant les droits d'appartenance du projet.
- manifest.yaml: Fichier manifeste principal du projet.
- README.md: Documentation du projet, fournissant des informations sur son utilisation, son installation et d'autres détails pertinents.
- values.yaml: Fichier de valeurs par défaut pour la configuration du chart.

***

## Chart

``` yaml
apiVersion: v2
```
Spécifie la version du format de chart Helm. Helm utilise différentes versions d'API pour assurer la compatibilité avec différentes versions de Helm.

``` yaml
name: .
```
Définit le nom du chart Helm. Dans ce cas, il semble être le répertoire actuel (indiqué par .). Le nom est un identifiant unique pour votre chart.

``` yaml
description: A Helm chart for Kubernetes
```
Fournit une brève description du chart Helm, expliquant son objectif. Cette description est généralement utilisée pour donner aux utilisateurs une idée de la fonctionnalité du chart.

``` yaml
type: application
```
Spécifie le type du chart Helm. Dans ce cas, il est marqué comme une application. Le type donne une indication sur la nature du chart, qu'il s'agisse d'une application, d'une bibliothèque ou d'un service.

``` yaml
version: 0.1.0
```
Indique la version du chart Helm lui-même. Il suit le format de version sémantique (Majeure.Mineure.Correction), et ce numéro de version est utilisé pour suivre les changements et les mises à jour du chart.

``` yaml
appVersion: "1.16.0"
```
Spécifie la version de l'application ou du service que le chart Helm déploie. Cela est distinct de la version du chart et permet de suivre la version de l'application déployée par le chart.

En résumé, le fichier chart.yaml fournit des métadonnées essentielles pour votre chart Helm, notamment son nom, sa description, son type, sa version et la version de l'application qu'il déploie. Il aide les utilisateurs de Helm et le système Helm à comprendre et à gérer le chart.


## Deployments

#### Vue d'ensemble
Ce code génère des définitions de déploiements Kubernetes en utilisant les valeurs fournies dans le fichier de configuration.

#### Itération sur les Déploiements
Le code utilise une boucle range pour itérer sur chaque déploiement défini dans le fichier de valeurs.

``` yaml
{{- range $key, $value := .Values.deployments }}
---
```

#### Définition du Déploiement
Chaque déploiement est défini en tant qu'objet Kubernetes de type "Deployment" (apps/v1).

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ $key }}
  labels:
    app: {{ $key }}
spec:
  ...
```

#### Configuration du Déploiement
Le bloc spec contient la configuration spécifique du déploiement, incluant le nombre de répliques, le sélecteur de pods, les conteneurs, les volumes, et les variables d'environnement.

``` yaml
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

- Répliques (replicas) : Définit le nombre de répliques pour le déploiement.
- Sélecteur de Pods (selector) : Spécifie le sélecteur de pods pour le déploiement.
- Conteneurs (containers) : Définit les conteneurs à exécuter dans les pods, incluant l'image, la politique de tirage d'image, les variables d'environnement, et les points de montage de volume.
- Volumes (volumes) : Définit les volumes à monter dans les pods, le cas échéant.
- Noms des conteneurs
- Les noms des conteneurs sont dérivés du nom du déploiement.

``` yaml
- name: {{ $key }}
```

#### Image du Conteneur
L'image du conteneur est déterminée en fonction des spécifications du déploiement, avec une logique conditionnelle pour le chemin du référentiel d'image.

``` yaml
image: "{{ $.Values.global.repository }}{{ $value.image.repository }}:{{ $value.image.tag | default $.Values.global.image.tag }}"
```

#### Variables d'Environnement
Les variables d'environnement sont définies en fonction des spécifications du déploiement et des variables globales.

``` yaml
envFrom:
  - name: env-secret
```

#### Gestion des ressources
Le bloc de configuration ci-dessous est utilisé pour définir les ressources Kubernetes (CPU et mémoire) pour les déploiements. Il utilise les valeurs fournies dans le fichier de configuration.

- Limites (limits) :
  - CPU : La quantité maximale de CPU qu'un conteneur peut utiliser.
  - Mémoire : La quantité maximale de mémoire qu'un conteneur peut utiliser.
- Demandes (requests) :
  - CPU : La quantité initiale de CPU que le conteneur demande.
  - Mémoire : La quantité initiale de mémoire que le conteneur demande.
Le code utilise des conditions pour vérifier si les valeurs limites et demandes sont définies dans le fichier de valeurs, et les utilise le cas échéant.

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

#### Points de Montage de Volume
Les points de montage de volume sont définis si spécifiés dans le déploiement.

``` yaml
volumeMounts:
  - mountPath: {{ $value.volume.mountPath }}
    name: {{ $key }}
```

#### Fin de la Boucle
La boucle range se termine à la fin du code.

``` yaml
{{- end }}
```

Cette configuration génère des déploiements Kubernetes basés sur les spécifications fournies dans le fichier de valeurs. Chaque déploiement a un nom et une configuration spécifiques, avec la possibilité de définir des répliques, des conteneurs, des volumes, et des variables d'environnement personnalisé.




## Services

#### Vue d'ensemble
Ce code génère des définitions de services Kubernetes en utilisant les valeurs fournies dans le fichier de configuration.

#### Itération sur les Services
Le code utilise une boucle range pour itérer sur chaque service défini dans le fichier de valeurs.

``` yaml
{{- range $key, $value := .Values.services }}
---
```

#### Définition du Service
Chaque service est défini en tant qu'objet Kubernetes de type "Service" (v1).

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: {{$.Values.global.name }}-{{ $key }}
  labels:
    app: {{ $key }}
spec:
  ...
```

#### Configuration du Service
Le bloc spec contient la configuration spécifique du service, incluant le type de service, l'adresse IP du cluster, les ports, et la sélection des pods associés.

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
- Type de Service (type) : Définit le type de service (LoadBalancer, ClusterIP, NodePort, etc.).
- Adresse IP du Cluster (clusterIP) : Spécifie l'adresse IP du service dans le cluster.
- Ports (ports) : Définit les ports du service, avec la possibilité de spécifier un ID et un NodePort pour chaque port.
- Sélecteur de Pods (selector) : Sélectionne les pods associés au service en utilisant une étiquette spécifique.

#### Noms des Ports
Les noms des ports sont dérivés du nom du service et, le cas échéant, de l'ID du port.
``` yaml
- name: {{ $key -}}-{{ $portValue.id -}}-service
```

#### Fin de la Boucle
La boucle range se termine à la fin du code.
``` yaml
{{- end }}`
```
Cette configuration génère des services Kubernetes basés sur les spécifications fournies dans le fichier de valeurs. Chaque service est nommé en fonction du nom global du service et du nom spécifique du service défini dans le fichier de valeurs.

## Persistant Volume Claim

Vue d'ensemble
Ce code génère des définitions de réclamations de volumes persistants (PVC) Kubernetes en utilisant les valeurs fournies dans le fichier de configuration.

Itération sur les Volumes Persistants
Le code utilise une boucle range pour itérer sur chaque volume persistant défini dans le fichier de valeurs.

``` yaml
{{- range $key, $value := .Values.volumes }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ $key }}
spec:
  ...
{{- end }}
```

#### Définition de la Réclamation de Volume Persistant
Chaque réclamation de volume persistant est définie en tant qu'objet Kubernetes de type "PersistentVolumeClaim" (v1).

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ $key }}
spec:
  ...
```

#### Configuration de la Réclamation de Volume Persistant
Le bloc `spec` contient la configuration spécifique de la réclamation de volume persistant, incluant les modes d'accès et les ressources demandées.

``` yaml
spec:
  accessModes: 
    ...
  resources:
    requests:
      ...
```

**Modes d'Accès (accessModes) :** Définit les modes d'accès au volume, en utilisant la valeur spécifiée dans le fichier de valeurs ou la valeur par défaut globale.

``` yaml
accessModes: 
  {{- if $value.accessModes }}
  - {{ $value.accessModes }}
  {{- else }}
  - {{ $.Values.global.volume.accessMode }}
  {{- end}}
```

**Ressources Demandées (resources.requests.storage) :** Définit la quantité de stockage demandée pour le volume, en utilisant la valeur spécifiée dans le fichier de valeurs ou la valeur par défaut globale.

``` yaml
resources:
  requests:
    {{- if $value.resources.requests.storage }}
    storage: {{ $value.resources.requests.storage }}
    {{- else }}
    storage: {{ $.Values.global.volume.storage }}
    {{- end}}
```

**Les sondes :**

Les sondes permettent de vérifier l'état d'un conteneur ou d'une application s'exécutant dans un pod.



#### Fin de la Boucle
La boucle `range` se termine à la fin du code.

``` yaml
{{- end }}
```

Cette configuration génère des réclamations de volumes persistants Kubernetes basées sur les spécifications fournies dans le fichier de valeurs. Chaque réclamation de volume persistant a un nom spécifique et une configuration détaillée, avec la possibilité de définir les modes d'accès et les ressources de stockage personnalisés.






