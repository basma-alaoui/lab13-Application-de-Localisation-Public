# lab13-Application-de-Localisation-Public

Présentation générale

MapApplication est une application Android qui permet de suivre la position GPS d’un utilisateur et de synchroniser ces données avec un serveur distant. La cartographie repose sur OSMDroid (OpenStreetMap) sans dépendance aux services Google. Le backend est écrit en PHP avec une base de données MySQL, et les communications utilisent la bibliothèque Volley. L’application gère les permissions dynamiques, optimise la batterie et identifie chaque appareil de manière unique via son ANDROID_ID.

Ce projet illustre l’intégration complète entre un client mobile et un serveur personnalisé pour la géolocalisation.


Fonctionnalités principales

- Suivi GPS précis : récupération des coordonnées (latitude, longitude, altitude, précision) via LocationManager.
- Synchronisation automatique : envoi périodique des positions au backend à l’aide de Volley.
- Visualisation cartographique : affichage de tous les points enregistrés sous forme de marqueurs sur une carte interactive OpenStreetMap.
- Gestion des permissions dynamiques conforme à Android 6.0 (API 23) et supérieur.
- Optimisation énergétique : le suivi GPS s’arrête automatiquement lorsque l’application passe en arrière-plan.
- Identification unique de l’appareil via ANDROID_ID pour associer les traces à un utilisateur.


Architecture technique

Client Android
- Langage : Java
- API minimale : 24 (Android 7.0)
- API cible : 36 (Android 15)
- Bibliothèques : OSMDroid pour la carte, Volley pour les requêtes HTTP, LocationManager pour le GPS.

Backend
- Serveur local ou distant (Apache, XAMPP, WAMP)
- PHP 7+ avec PDO pour l’accès à la base de données
- MySQL : base de données nommée map_project contenant une table positions (id, android_id, latitude, longitude, altitude, accuracy, timestamp)

Communication
- Les positions sont envoyées au serveur via une requête POST (script PHP).
- Le client peut également récupérer l’historique des positions d’un appareil pour les afficher sur la carte.


Installation et configuration

Étape 1 – Préparer le serveur backend

- Installer un serveur local (XAMPP, WAMP, MAMP) ou utiliser un hébergement distant.
- Démarrer Apache et MySQL.
- Accéder à phpMyAdmin.
- Créer une base de données nommée map_project.
- Importer le fichier SQL fourni dans le dossier backend_scripts/database_setup.sql.
- Copier l’ensemble des fichiers PHP du dossier backend_scripts dans le répertoire racine du serveur (htdocs pour XAMPP, www pour WAMP).

Étape 2 – Adapter l’URL du serveur dans l’application

Ouvrir les fichiers MainActivity.java et GoogleMapActivity.java. Rechercher la variable contenant l’URL du serveur. Remplacer par l’adresse correcte :
- Pour un émulateur Android : utiliser 10.0.2.2 (qui correspond à localhost de la machine hôte).
- Pour un appareil physique : utiliser l’adresse IP locale du serveur (exemple : 192.168.1.x).

Exemple :
String serverUrl = "http://10.0.2.2/map_project/save_location.php";

Étape 3 – Ouvrir et compiler le projet Android

- Lancer Android Studio.
- Ouvrir le dossier du projet MapApplication.
- Laisser Gradle se synchroniser (vérifier la connexion internet pour télécharger les dépendances OSMDroid et Volley).
- Connecter un appareil Android (ou démarrer un émulateur avec Google APIs et support GPS).
- Cliquer sur Run (triangle vert).


Utilisation de l’application

1. Au premier lancement, l’application demande les permissions de localisation (fine location). Accepter.
2. Le suivi GPS démarre automatiquement. Un message Toast confirme régulièrement l’envoi des coordonnées au serveur.
3. Pour visualiser les positions enregistrées sur une carte, appuyer sur le bouton "Afficher La Map" (ou équivalent selon l’interface).
4. La carte se charge et affiche des marqueurs à chaque position historique de l’appareil. Il est possible de zoomer et de naviguer.
5. Lorsque l’application passe en arrière-plan (appui sur Accueil ou passage à une autre app), le suivi GPS s’arrête automatiquement pour économiser la batterie. Il reprend au retour au premier plan.


Tests et validation

- Sur émulateur : utiliser l’outil de simulation de localisation (Extended Controls → Location) pour envoyer des coordonnées fictives. Vérifier que les points apparaissent dans la base de données et sur la carte.
- Sur appareil réel : se déplacer à l’extérieur (ou laisser l’appareil fixe) et observer les remontées dans la console Logcat (filtre "Volley" ou "Location").
- Vérifier l’absence d’erreur UnsatisfiedLinkError (OSMDroid ne nécessite pas de code natif).
- Tester la rotation de l’écran : la carte doit conserver son état (centrage et marqueurs) grâce à la sauvegarde d’instance.


Structure du code (aperçu)

- MainActivity.java : activité principale qui gère le LocationManager, l’envoi des positions avec Volley, et les permissions.
- GoogleMapActivity.java : activité dédiée à l’affichage de la carte avec OSMDroid. Récupère l’historique des positions depuis le serveur.
- Fichiers backend (PHP) :
  - save_location.php : reçoit les données POST (android_id, lat, lon, alt, acc) et les insère en base.
  - get_locations.php : retourne la liste des positions d’un android_id donné au format JSON.
- layout XML : activity_main.xml et activity_map.xml définissent les interfaces (bouton, conteneur de carte).


Optimisations et bonnes pratiques

- Libération du LocationManager dans onPause() pour éviter une consommation inutile.
- Utilisation de Volley en file d’attente (RequestQueue) pour ne pas saturer le réseau.
- Gestion des erreurs réseau : les tentatives d’envoi échouées sont ignorées (pas de file d’attente persistante – à améliorer selon les besoins).
- Identifiant unique : ANDROID_ID est stable pour un appareil donné (attention aux restrictions Android 8.0+ sur certains fabricants – solution alternative possible).
- La carte OSMDroid utilise un cache des tuiles pour réduire la consommation de données.


Limites connues

- L’application ne stocke pas les positions en local si le réseau est indisponible (elles sont perdues).
- Le suivi GPS en arrière-plan n’est pas implémenté pour rester dans les limites de l’optimisation batterie et des politiques récentes d’Android (foreground service nécessaire pour un suivi continu).
- Les performances sur des traces très nombreuses (plusieurs milliers de points) peuvent ralentir l’affichage des marqueurs (à optimiser par clustering).


Améliorations possibles

- Ajouter un service foreground avec notification pour un suivi continu en arrière-plan.
- Stocker localement (SQLite ou Room) les positions non synchronisées et les renvoyer ultérieurement.
- Filtrer les positions par date et afficher un tracé (polyline) au lieu de simples marqueurs.
- Implémenter l’authentification des utilisateurs au lieu d’utiliser uniquement l’ANDROID_ID.


Conclusion

MapApplication est un exemple complet d’application de géolocalisation avec son propre backend. Elle couvre les aspects essentiels : acquisition GPS, communication réseau, affichage cartographique et gestion du cycle de vie. Ce projet peut servir de base à des applications plus avancées comme le suivi de flotte, le partage de position ou le journal de déplacement.

Pour toute question ou contribution, se référer à la documentation fournie dans les commentaires du code ou ouvrir une issue sur le dépôt du projet.
