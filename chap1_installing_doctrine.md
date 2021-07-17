# Installing Doctrine

Symfony ne dispose pas de lui-même d'une couche de base de données.  
Pour gérer une base de données, on va donc faire appel à une bibliothèque externe à Symfony, appelée **Doctrine**.
Pour info, le projet qu'on va utiliser pour suivre le tuto est dans `code-symfony4-doctrine/start/`.  

Pour installer Doctrine, on exécute la commande `composer require doctrine`.

La configuration de la base de données est accessible dans le fichier `.env`.  
On peut déjà y voir que la `DATABASE_URL` a été ajoutée dans ce fichier via la recipe `DoctrineBundle`.  

On l'adapte à notre base de données locale: `DATABASE_URL="mysql://root:rootst@127.0.0.1:3306/the_spacebar"`.  

Cette variable d'environnement est utilisée dans le fichier `config/packages/doctrine.yaml`, sous doctrine > dbal > url.  
La commande `resolve` est pour rappel expliquée dans le chapitre 16 de la partie 2.  
Ce fichier de configuration permet par exemple de mettre toutes les tables en UTF-8, ou d'utiliser systématiquement des underscores pour les noms 
de table et colonnes. Le numéro de version précisé doit être celui utilisé en production.  

Une fois ce fichier en place, on peut alors créer notre base de données (vide) avec la commande `php bin/console doctrine:database:create`.

Attention, en MySQL 8.0 on doit entrer la requête `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '';`
pour que la commande de doctrine marche.
On peut vérifier la version actuelle de MySQL via la commande `mysql -V`.