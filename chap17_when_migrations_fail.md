# When Migrations Fail

On va ajouter deux champs `createdAt` et `updatedAt` dans notre Articles, comme d'habitude via la commande 
`php bin/console make:entity`, puis `php bin/console make:migration` et `php bin/console doctrine:migrations:migrate`.
Sauf que la migration a échoué : il y a des cas où quand on ajoute une colonne NOT NULL à une table ayant déjà 
des données, MySQL a du mal à savoir quelles données mettre aux valeurs déjà existantes.

Pour éviter de devoir purger la base, on va d'abord créer les nouveaux champs pouvant être nuls, puis mettre la date
d'aujourd'hui dans les données existantes avant de rendre la colonne obligatoire.  
Techniquement, on va modifier "à la main" le dernier fichier créé dans le dossier `migrations/`, puis refaire un
`php bin/console doctrine:migrations:migrate`.
Puis pour rendre les deux nouvelles colonnes obligatoires, il suffit de refaire `php bin/console make:migration`.  
Un nouveau fichier de migration est créé pour vraiment rendre les deux nouveaux champs obligatoires.
On peut alors refaire `php bin/console doctrine:migrations:migrate`, et le tour est joué.

Attention, il y a des cas où la migration ne va pas réussir si elle est sur plusieurs lignes, que 1 a réussi mais pas toutes 
les lignes. Si aucun bricolage dans le fichier dans `migrations` n'est possible, on doit parfois drop toute la base de données
et recommencer. Pour ce faire, on doit d'abord exécuter `bin/console doctrine:database:drop --force`, puis
`bin/console doctrine:database:create`.

