# TP 4.1 — Externaliser la configuration avec Kubernetes

## 1. Objectif du TP

L'objectif de ce TP est de séparer la configuration d'une application de son image Docker en utilisant les ressources Kubernetes suivantes :

- **Namespace** : isoler les ressources du TP.
- **ConfigMap** : stocker les paramètres non confidentiels.
- **Secret** : stocker les informations sensibles, comme les identifiants.
- **Deployment et Pod** : déployer l'application et lui fournir la configuration nécessaire.

## 2. Environnement utilisé

- Kubernetes : v1.34.12
- Système : Ubuntu Server 26.04.1 LTS
- Runtime : containerd
- Réseau du cluster : Calico
- Namespace : `tp-k8s`
- Image de l'application : `nginx:1.26-alpine`

## 3. Étapes réalisées

### Étape 1 — Création du Namespace

Création d'un espace de noms dédié au TP :

```bash
kubectl create namespace tp-k8s
```

Vérification :

```bash
kubectl get namespaces
```

### Étape 2 — Création du ConfigMap

Création du ConfigMap `app-config` pour externaliser les paramètres non confidentiels de l'application.

```bash
kubectl get configmap app-config -n tp-k8s
```

Le ConfigMap permet de séparer les paramètres de configuration de l'image de l'application.

### Étape 3 — Création du Secret

Création du Secret `db-credentials` pour stocker les identifiants de l'application.

```bash
kubectl create secret generic db-credentials \
  --from-literal=DB_USER=appuser \
  --from-literal=DB_PASSWORD='<mot-de-passe>'
```

**Remarque :** remplacer `<mot-de-passe>` par une valeur choisie lors de l'exécution. Ne jamais publier un véritable mot de passe dans GitHub.

Vérification :

```bash
kubectl get secret db-credentials -n tp-k8s
```

### Étape 4 — Déploiement de l'application

Le Deployment `demo` utilise l'image `nginx:1.26-alpine` et référence les ressources `app-config` et `db-credentials`.

Vérification des Pods :

```bash
kubectl get pods -n tp-k8s -o wide
```

Inspection du Pod :

```bash
kubectl describe pod <nom-du-pod-demo> -n tp-k8s
```

Les volumes `config` et `secret` apparaissent dans la description du Pod, confirmant que les deux ressources sont référencées.

### Étape 5 — Vérification du fonctionnement

Le Pod `demo` a été créé et exécuté avec succès.

Résultat observé :

- Le Pod est passé à l'état `Running`.
- Le conteneur est devenu `Ready`.
- Le Pod a reçu une adresse IP.
- Les références au ConfigMap et au Secret sont présentes.
- Après la suppression du Pod de test, le Deployment a créé un nouveau Pod automatiquement.

Commande de vérification :

```bash
kubectl get pods -n tp-k8s -o wide
```

## 4. Difficulté rencontrée

Pendant le déploiement, le message suivant est apparu :

```text
FailedCreatePodSandBox
plugin type="calico" failed (add):
error getting ClusterInformation:
connection is unauthorized: Unauthorized
```

Après la recréation du Pod `calico-node` sur le worker `w1`, puis celle du Pod `demo`, l'application a démarré correctement.

La gestion durable des identifiants CNI de Calico reste à surveiller pour éviter la réapparition de cette erreur.

## 5. Résultat final

Le TP a permis de mettre en pratique :

- La création et l'utilisation d'un Namespace.
- L'externalisation de la configuration avec un ConfigMap.
- La gestion des informations sensibles avec un Secret.
- L'utilisation de ces ressources dans un Pod Kubernetes.
- La vérification de l'état d'un déploiement avec `kubectl`.

**Statut :** déploiement fonctionnel et Pod `demo` en état `Running` et `Ready` lors de la dernière vérification.

## 6. Références

- [Documentation officielle Kubernetes — ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Documentation officielle Kubernetes — Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

