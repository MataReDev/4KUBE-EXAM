# 4KUBE-EXAM

Pour ce projet nous devions mettre en place application distribuée permettant de suivre en temps réel une flotte de véhicules effectuant des livraisons.

Pour cela nous devions :
 1) Créer un déploiement pour chaque conteneur.

 2) Créer un service interne pour chaque déploiement.

 3) Utiliser un volume pour la base de données.<br>




Pour commencer, voici quelques commandes utiles à la création des conteneurs :<br>

Commande pour générer le fichier manifest.yaml qui sert a lancer les pods :
`helm template . > manifest.yaml`

Commande pour installer le fichier manifest et créer les pods :
`helm install kubeexam .`

Commande pour afficher la liste des pods créés :
`kubectl get pods`

Commande pour afficher les infos du pod sélectionné :
`kubectl describe pod [idPod]`