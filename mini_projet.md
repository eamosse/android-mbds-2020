# Mini projet Android (MBDS 2020) 

Mise en place d'une application Android newsletter (MBDS News) exploitant les données du site https://newsapi.org/. 

> Instructions pour démarer le projet 
>>[Cliquez ici](/project_instructions.md)

## Fonctionnalités à developper 
Les principales fonctionnalités à implémenter incluent (mais ne se limitent pas seulement à :

### Page d'accueil de l'application
La mise en place de la page d'accueil de l'application dans laquelle les options de consulation sont organisées :

- Par éditeurs (https://newsapi.org/docs/endpoints/sources)
- Par catégories: Politique, Economie, Education, Pandémie,
- Par pays : France, Chine, Etats Unis, Angleterre….

### Liste des articles par catégorie
Quand l'utilisateur selectionne un article sur la page d'accueil, afficher tous les articles correspondant à cette catégorie dans un fragment. 

La vue doit contenir : 
 - Le titre de l'article, 
 - un appercu du contenu du message (3 lignes maximum)
 - L'auteur de l'article
 - La date de publication 
 - L'image liée à l'article
 - un bouton permettant d'ajouter un article en favori (un coeur). Le coeur creux si l'article n'est pas en favori, sinon il est plein.  

### Gestion des favoris 
Quand l'utilisateur clique sur l'icone favori, il faut sauvegarder l'article dans une base de données locales.

Définir une vue permettant à l'utilisateur de voir tous ses articles favoris. Le lien vers cette vue doit être placée dans la barre de navigation en haut à droite dans la page d'accueil et dans la la vue liste d'articles. 

### Détail d'un article
Quand l'utilisateur clique sur un article, créer une vue permettant d'afficher le détail de l'article. Cette vue doit afficher toutes les informations de l'article ainsi qu'un lien vers le site de l'éditeur. 
Le lien doit ouvrir une page web dans un navigateur externe.

> Ajouter une option dans la barre d'action permettant à l'utilisateur d'ajouter en favori l'article qu'il est entrain de visualiser. 

### Vue à propos de nous
Ajouter une vue permettant d'afficher des détails sur l'application. Cette vue doit afficher : 

- Les noms et prénoms des membres du groupe (une photo aussi si vous le voulez)
- Un lien vers le repo du projet (sur Github)
- La liste des librairies externes utilisées dans l'applications
- La liste des fonctionnalités de l'application 

Ajouter une option en haut à droite de la toolbar permettant d'afficher la vue à propos de nous. 

## Contraintes 
- Le projet doit être developpé en Android natif 
- Groupes de 3 personnes
    - pas de personnes seules 
    - un groupe de 2 ou de 4 si le nombre d'étudiants n'est pas multiple de 3
- Prioriser le langage Kotlin 
- Les vues doivent être developpées en XML
- Suivre le Design Pattern MVC
    - Utiliser les package pour regrouper les composants du projet
- N'utiliser qu'une seule activité
> Attention, l'activité doit gérer uniquement les parties communes entre les différentes vues (comme la toolbar) et la gestion des fragments. 
- Utiliser git pour la gestion des versions
> Je me baserait sur les commits pour controler le travail de chacun. Cela dit, il n'y aura pas une note globale pour le groupe. 


## Modalités de rendus 

- Date limite : 22 Novembre 2020
- Livrables 
    - Envoyer un mail à eamosseatgmaildotcom avec le lien vers le repo du projet ainsi que les noms des membres du groupe

## Coaching ? 
Si vous avez besoin d'aide sur un sujet bloquant ou si vous avez un doute sur quoi que ce soit, n'hésitez pas à m'envoyer un mail. 
