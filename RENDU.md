# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : FAKOREDE-KOUCHADE**
**Groupe : RS2**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > 3 nœuds. k3d-lab-agent-0 est le controle-plane. k3d-lab-agent-0 et k3d-lab-agent-1 sont les workers.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > docker ps affiche 4 conteneurs. k3d-lab-server-0, k3d-lab-agent-0 et k3d-lab-agent-1 sont les images de rancher/k3s:v1.35.5-k3s1. k3d-lab-serverlb est le load balancer. Un nœud dans k3d est un conteneur Docker qui joue le rôle d’une machine en faisant tourner k3s.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > kubectl create deployment a créé un Deployment (deployment.apps/web), qui crée un ReplicaSet (replicaset.apps/web-68d995574f), qui crée à son tour le Pod (pod/web-68d995574f-slkc5).
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Le pod web-68d995574f-zdsl8 a été supprimé, et un nouveau pod web-68d995574f-slkc5 a été créé automatiquement.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non. La modification a été faite dans le système de fichiers du conteneur, qui est éphémère. Le nouveau pod repart de l'image d'origine nginx, qui ne contient pas le changement.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Quand le pod a été supprimé, la cible du tunnel n'existait plus et la connexion a été cassé.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > kubectl create crée un objet une seule fois : si on relance, erreur car il existe déjà. kubectl apply part d’un fichier qui décrit ce qu’on veut : il crée l’objet s’il n’existe pas, ou le met à jour s’il existe. On peut le relancer autant de fois qu’on veut.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le selector dit au Service à quels pods envoyer le trafic, grâce à leur étiquette (app: site). S’il ne correspond à aucun pod, le Service n’a personne à qui envoyer le trafic, et le site ne répond plus.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
```
NAME                    READY   STATUS    RESTARTS   AGE   IP          NODE               NOMINATED NODE   READINESS GATES
site-5c96d45fb8-h8pv4   1/1     Running   0          20m   10.42.1.6   k3d-lab-agent-1    <none>           <none>
site-5c96d45fb8-s8t85   1/1     Running   0          15m   10.42.2.7   k3d-lab-server-0   <none>           <none>
site-5c96d45fb8-wn6w8   1/1     Running   0          15m   10.42.0.7   k3d-lab-agent-0    <none>           <none>
```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le pod était en ImagePullBackOff. La commande kubectl describe pod a montré la cause : la version nginx:9.99 n’existe pas, donc Kubernetes ne peut pas télécharger l’image.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui. Kubernetes garde les anciens pods tant que le nouveau ne marche pas, donc ils ont continué à faire tourner le site.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://sturdy-journey-4xgqvpvq9v4c747v-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Avec kubectl get deploy mario, on retrouve bien le Deployment. C’est un Deployment normal, comme ceux créés en ligne de commande. Portainer ajoute juste quelques labels à lui. Portainer et kubectl parlent au même composant : l’API server de Kubernetes. Portainer est juste une interface graphique par-dessus.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > Bonus A, B, C et D réalisés.
    > A : le site marche sur le port 8081, même après suppression des pods.
    > B : readinessProbe et limites CPU/mémoire ajoutées. Avec `/nope`, le pod n'est jamais prêt et ne reçoit pas de trafic.
    > C : après le `drain`, les pods vont sur les autres nœuds. Après le `uncordon`, ils ne reviennent pas tout seuls.
    > D : une deuxième copie du site tourne dans le namespace `prod`, séparée de `default`.
    > Remarque : Portainer demandait un token, trouvé dans ses logs.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18
