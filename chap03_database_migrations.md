# Database Migrations

Pour créer la table associée à l'entité créée dans le chapitre précédent, on applique dans un premier temps la commande
`./bin/console make:migration`. Cette commande a créé le fichier "Version202106561616161.php" dans le dossier 
`migrations/`. On peut constater dans ce fichier, comme vu à Label Emmaüs, que le fichier contenant les requêtes SQL
associées à la création de l'entité ont bien été créées.  
Pour exécuter les requêtes SQL associées à ce fichier de migration, on applique la commande `./bin/console doctrine:migrations:migrate`.  
Si les requêtes ont déjà été exécutées (si on exécute deux fois cette même commande d'affilée par exemple), aucune 
requête n'est exécutée.  

En exécutant la commande `./bin/console doctrine:migrations:status`, on peut voir que le système de migrations de Doctrine 
trace les migrations effectuées à l'aide de la table `migration_versions`. Si les migrations du dossier `migrations/` ont déjà 
toutes été présentes dans cette table, alors le système de migrations ne mettra pas à jour la bdd.  

Le serveur de production aura sa propre table `migration_versions`: s'il y a eu de nouvelles modifications du MCD depuis 
la dernière MEP, alors le système de migrations saura prendre en compte les nouvelles migrations qui ont été créées.  

On souhaite à présent modifier notre Entité a posteriori : on veut que le champ "slug" soit unique.  
Pour ce faire, on va ajouter dans l'annotation `@ORM\Column` le paramètre `unique=true`. 

Pour que cette modification soit prise en compte, on applique donc la commande `./bin/console make:migration` (où on note
qu'on peut écrire `migratio` au lieu de `migration` et la commande est tout de même exécutée par Symfony tant qu'il n'y a pas 
d'ambiguïté). On peut voir qu'un nouveau fichier de migrations a été créé, ajoutant l'unique index à la table (qui est la seule 
différence par rapport à la migration précédente).  
Puis rebelote: l'index unique est ajouté en bdd via un `./bin/console doctrine:migrations:migrate`.