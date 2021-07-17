# Activating Timestampable

En ajoutant `timestampable:true` , dans `stof_doctrine_extensions.yaml`, puis les annotations Gedmo suivantes
dans l'entité de l'Article :
```PHP
/**
 * @ORM\Column(type="datetime")
 * @Gedmo\Timestampable(on="create")
 */
private $createdAt;

/**
 * @ORM\Column(type="datetime")
 * @Gedmo\Timestampable(on="update")
 */
private $updatedAt;
```

Nos deux nouveaux champs `createdAt` et `updatedAt` créés dans le chapitre précédent vont être mis à jour automatiquement 
lors de la création ou de la mise à jour d'un article, sans avoir besoin de le faire nous-même.
Après un `./bin/console doctrine:fixtures:load`, puis un `./bin/console doctrine:query:sql 'Select * from article'`, on peut 
voir les dates de création et de mise à jour des articles bien mises automatiquement en base.  

Mais il existe en fait un bien meilleur moyen d'avoir ces deux nouveaux champs : on peut directement supprimer ces 
deux attributs et leurs accesseurs de la classe `Article`, pour à la place mettre :
```PHP
class Article
{
    use TimestampableEntity;
        ...
}
```

Et en allant faire un tour (`Ctrl + Click`) dans la `TimestampableEntity`, on peut y voir les deux champs `createdAt`
et `updatedAt` qu'on avait écrits à la main auparavant.  

Même après avoir modifié l'Entité de cette manière, la commande `./bin/console make:migration` indique qu'aucun changement 
n'a besoin d'être fait en base de données. De même, les commandes `./bin/console doctrine:fixtures:load` et
`./bin/console doctrine:query:sql 'Select * from article'` montrent que tout se passe exactement comme avant, même après
avoir changé le code pour mettre notre raccourci.

La Recipe doctrine est fournie avec son propre fichier de config `config/packages/prod/doctrine.yaml`. Ce fichier 
indique en gros que tout ce qui peut être mis en cache facilement est mis en cache.  
Ce fichier porte aussi sur des trucs de relation, qui vont être vus dans la Partie 4.