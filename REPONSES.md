# TP4 — Dépôts d'artefacts avec Nexus

## Partie 1 — Les dépôts npm

### Q1. Quelle est la différence entre un dépôt proxy, hosted et group ?

- **Proxy :** récupère les paquets depuis un registre externe, comme npmjs.org, et les met en cache.
- **Hosted :** stocke les paquets créés et publiés par notre équipe.
- **Group :** regroupe plusieurs dépôts sous une seule adresse pour faciliter la lecture.

### Q2. Quel est le rôle de npm-proxy ?

npm-proxy permet de récupérer les dépendances depuis le registre public npmjs.org et de les mettre en cache dans Nexus. Cela permet d'améliorer les performances et de limiter les accès directs à Internet.

### Q3. Pourquoi utilise-t-on npm-group pour installer les dépendances et npm-hosted pour publier un paquet ?

On utilise npm-group pour installer les dépendances, car il regroupe les dépôts hosted et proxy dans une seule adresse.

On utilise npm-hosted pour publier nos propres paquets, car un dépôt group est destiné à la lecture et non à la publication.

## Partie 2 — Consommer via le group

### Q4. Pourquoi utilise-t-on nexus au lieu de localhost dans le fichier .npmrc ?

On utilise nexus car npm s'exécute dans un conteneur Docker. Dans le réseau Docker Compose, nexus est le nom du service permettant aux conteneurs de communiquer entre eux.

### Q5. Quel est le rôle du fichier frontend/.npmrc ?

Il permet de configurer le registre npm utilisé par le frontend. Il redirige les téléchargements des dépendances vers npm-group de Nexus.

### Q6. Pourquoi le fichier package-lock.json ne doit-il pas changer après la configuration de Nexus ?

Parce que les versions des dépendances restent identiques. Seule la source de téléchargement change : les dépendances passent désormais par Nexus au lieu d'être téléchargées directement depuis npmjs.org.

## Partie 3 — Publier un paquet interne

### Q7. Quel est le rôle du paquet @devops2/cours-utils ?

Ce paquet contient les fonctions réutilisables liées à la gestion des cours, comme formatDuration, filterCours et sortBySession. Il permet de partager cette logique entre plusieurs applications.

### Q8. Que signifie la version 0.1.0 selon SemVer ?

La version 0.1.0 signifie :
- 0 : version majeure, développement initial.
- 1 : version mineure.
- 0 : version corrective.

Le format SemVer est MAJEUR.MINEUR.CORRECTIF.

### Q9. Pourquoi Nexus refuse-t-il de republier la version 0.1.0 ? Quelle est la politique utilisée ?

Nexus refuse la republication car le dépôt npm-hosted utilise la politique Disable redeploy.

Cette politique interdit de remplacer une version déjà publiée afin de garantir la stabilité et l'intégrité des paquets.

### Q10. Quelle doit être la nouvelle version après l'ajout de la fonction formatLevel ?

La nouvelle version doit être **0.2.0**, car on ajoute une nouvelle fonctionnalité compatible avec les fonctionnalités existantes.

Il s'agit d'une augmentation de la version mineure.

### Q11. Pourquoi le frontend utilise-t-il @devops2/cours-utils au lieu de @/lib/cours ?

Le frontend utilise désormais le paquet interne publié dans Nexus afin de centraliser la logique métier et de faciliter sa réutilisation et sa maintenance.

Cela évite également de dupliquer le même code dans plusieurs applications.

## Partie 4 — L'image Docker

### Q12. Quelle est la différence entre docker-hosted, docker-proxy et docker-group ?

- **docker-hosted :** stocke les images Docker publiées par notre équipe.
- **docker-proxy :** récupère et met en cache les images provenant de Docker Hub.
- **docker-group :** regroupe docker-hosted et docker-proxy pour permettre de récupérer les images depuis une seule adresse.

### Q13. Quelle est la différence entre un tag et un digest Docker ?

Un tag est un nom attribué à une image, par exemple 1.0.0. Il permet d'identifier une version de l'image.

Un digest est une empreinte SHA256 unique qui identifie précisément le contenu de l'image.

Contrairement au tag, le digest est immuable.

### Q14. Pourquoi le digest après le push et le pull doit-il être identique ?

Le digest doit être identique car il représente le contenu exact de l'image Docker.

Si le digest est identique après le push vers docker-hosted et le pull depuis docker-group, cela prouve que l'image récupérée correspond exactement à celle qui a été publiée.

### Q15. Pourquoi utilise-t-on docker-group pour récupérer les images et docker-hosted pour publier ?

On utilise docker-group pour récupérer les images, car il regroupe les images internes et celles provenant de Docker Hub.

On utilise docker-hosted pour publier les images créées par notre équipe.

Cela respecte la règle : on lit via le group et on écrit dans le hosted.

## Partie 5 — Secrets et synthèse

### Q16. Pourquoi ne faut-il jamais commiter les secrets dans Git ?

Il ne faut jamais commiter les secrets dans Git car ils peuvent être consultés par les personnes ayant accès au dépôt.

Même si un secret est supprimé dans un commit ultérieur, il reste présent dans l'historique Git.

Pour protéger les informations sensibles, on utilise le fichier .env, qui est ajouté dans .gitignore.

Les fichiers .npmrc contiennent uniquement une référence à la variable d'environnement ${NPM_AUTH}, et non la valeur réelle du secret.