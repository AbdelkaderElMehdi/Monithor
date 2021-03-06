# Documentation projet
## Problématique
Un pod avec un spring boot en marche éxpose plusieurs EndPoints d'API-REST, On peut monitorer l'état de toutes les repliques d'un pods ( voire si ils sont en marche ou pas), mais on ne peut pas connaitre l'état de fonctionnement de chaque End-Point surtout si celui-ci possede des dépendances (tel que l'appel à une BD, Queue, Systeme de fichier, Cache partagé, etc).
En d'autres termes, un pods peut fonctionner normalement (répends positif sur kubernetes et sur les logs), mais un ou plusieurs de ses End-Points peuvent être HS du au dépendances de celles-ci qui sont HS.

Exemple : 
- EndPoint 1 de l'Api 1 dépend uniquement de MongoDB (Fonctionne normalement).
- EndPoint 2 de l'Api 1 depend de MongoDB mais aussi d'un EndPoint 3 de l'Api 2 qui lui même dépend d'autre Apis ... ( ne fonctionne pas, comment savoir ou se trouve l'anomalie ?).

## Objectif

le but de ce systeme de monitoring est de :

- Construire un arbre de dépendances pour chaque endPoint et de monitorer son état en temps réel.
- Fournir un portail avec tous les endPoints de chaque domaine et leurs état (en vert pour en marche et rouge pour HS).
- Pour chaque EndPoint, permettre la visualisation de l'arbre des dependances (avec leurs état encore une fois).
- Proposer des logs centralisées :
  - Pour chaque crash : identifier la dépendance qui crache.
  - Envoyer un log global avec tout les états des dependances.

## Difficultés possible
Comment connaitre les dependances de chaque EndPoints: 
- Coder en brute force dans chaque domaine un endpoint " GET /dependencies " qui éxpose pour chaque EndPoint les dependence en clair.
- Utiliser les annotations pour générer automatiquement un EndPoint. ( AOP ? AspectJ?).
- Utiliser les Queue de RabbitMQ pour identifier les dependances.

Trouver une librairie pour la visualisation de Graphs (Grossomodo pour afficher un arbre avec differents neouds).

## Technologies 
- SpringBoot pour le coté serveur (+ Maven ) ( + Aop???) ( Actuator ?).
- Angular pour le client. (+ redux)
- [Vis.js Networks](https://visjs.github.io/vis-network/docs/network/) pour visualiser les graphs et reseaux de dependances.
- Elasticsearch/MongoDB pour le stockage ??
- Kibana/Grafana ??
- La mitraille DevOps ( jenkins, sonarQube, Selenium, Docker, kubernetes, helm).

## Architecture de base (en évolution)
L'application sera découpé en :
- Une application SpringBoot qui intéroge les differentes Apis et EndPoints avec un flux websocket pour communiquer au client les logs.
- Un client Web Angular-8 avec Vis.js pour visualiser les Graphs de dépendances et l'états de tout les endPoints.
- Une BD Mongo ou ElasticSearch pour centraliser et stocker les logs d'états des endpoints du cluster.

![60% center](https://github.com/AbdelkaderElMehdi/Monithor/blob/master/Diagrammedeclasses.png "Diagramme de classe")

Le Model de données de l'application est structure comme suit :
- Tout est une ressource : l'appel à un endPoint d'Api-REST ou à un service de stockage de données, ces ressource sont identifié par leurs types respectif (EndPoint ou un accés à MongoDB ... RabbitMQ), pour chaque ressource, elle possede :
  - un url..
  - un port.
  - un nom.
  - un état pour en marche ou pas.
  - un ensemble de dépendances qui sont elles-mêmes des ressources.
  - un id calculé en fonction d'un hash d'attribut pour faciliter la définition du `.equals()` .
- Chaque ressource possede quelque méthodes clés :
  - `getLifeState() ` : Permet de faire un appel récursif vers l'ensemble des dependance pour avoir leurs lifeState et faire un Et-Logique avec la totalité de l'arbre.
  - `resolveDependencies` : Permet dans le cas d'un probleme de retourner une liste des ressources qui crachent.



