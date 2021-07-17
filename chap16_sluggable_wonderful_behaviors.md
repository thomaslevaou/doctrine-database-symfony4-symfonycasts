# Sluggable & other Wonderful Behaviors

La création de slug étant à présent faite via `$this->faker->slug`, les slugs des articles sont à présent
des chaînes de caractères aléatoires.  
Mais on aimerait que ce soient des chaînes de caractères ressemblant au titre de l'article, mais sans avoir à les 
écrire manuellement.  

Ceci est déjà permis en PHP, et le bundle qui facilite l'intégration de cela en Symfony s'appelle **StofDoctrineExtensionsBundle**.  

On l'installe via la commande `composer require stof/doctrine-extensions-bundle`.  
Notons qu'en installant via commande, Symfony nous demande si on veut installer la recipe du `StofDoctrineExtensionsBundle`.  
Si Symfony nous demande une confirmation ici alors qu'habituellement non, c'est en raison du fait que cette recipe 
vient du repo "contrib" (à la différence des autres recipes qui viennent du "main" repository est dont la qualité est 
plus fortement contrôlée), et où grosso modo n'importe qui peut contribuer. D'où le fait que Symfony demande une confirmation 
ici, par mesure de sécurité.  

Cette Recipe a installé un nouveau fichier `stof_doctrine_extensions.yaml` dans `config/packages/`.  
Le contenu de ce fichier doit être modifié pour devenir le suivant :
```YAML
# Read the documentation: https://symfony.com/doc/current/bundles/StofDoctrineExtensionsBundle/index.html
# See the official DoctrineExtensions documentation for more details: https://github.com/Atlantic18/DoctrineExtensions/tree/master/doc/
stof_doctrine_extensions:
    default_locale: en_US
    orm:
        default:
            sluggable: true
```

Puis dans l'Entité, une annotation supplémentaire doit être ajoutée au-dessus de la déclaration du `slug` :
```PHP
/**
 * @ORM\Column(type="string", length=100, unique=true)
 * @Gedmo\Slug(fields={"title"})
 */
private $slug;
```

Et c'est tout pour générer des slugs ! On peut supprimer la ligne `->setSlug($this->faker->slug)` dans `ArticleFixtures`,
les slugs sont à présent générés automatiquement par les lignes qu'on vient d'écrire.  
En exécutant de nouveau `php bin/console doctrine:fixtures:load`, on peut bien constater que les noms des slugs sont à 
présents proches des noms des articles.  
Certains slugs ont un nombre à la fin, pour s'assurer que tous les slugs sont uniques.  

Si un jour on doit mettre à jour un index ou un autre truc automatiquement à chaque création, modification ou suppression 
d'entité, on peut mettre en place un système de "Doctrine Event Subscriber", disponible dans la documentation.