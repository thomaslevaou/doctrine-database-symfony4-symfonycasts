# Fun with Twig Extensions!

Le chapitre précédent a fait basculer la traduction du markdown dans la vue Twig `show.html.twig` via `|markdown`. De ce 
fait, nous n'utilisons plus le MarkdownHelper qui stockait les markdowns déjà traduits en cache.  
On souhaite à présent pouvoir utiliser un service créé par nous comme `MarkdownHelper` dans une vue Twig.  

Pour ce faire on va créer notre propre **filtre Twig**. L'appellation de "filtre" est la même qu'en Vue.js.  

Et pour créer notre propre extension Twig, on va appliquer la commande `./bin/console make:twig-extensions`.  
On va d'abord créer une extension `AppExtension` qui va contenir le code générique à toutes les extensions Twig.  
Ce fichier a été créé dans `src/Twig/AppExtension.php`.

Juste histoire de tester le bon fonctionnement du filtre, on va écrire le code suivant qui va faire passer la chaîne filtrée
en majuscules :
```PHP
class AppExtension extends AbstractExtension
{
    public function getFilters(): array
    {
        return [
            new TwigFilter('cached_markdown', [$this, 'processMarkdown'], ['is_safe' => ['html']]),
        ];
    }

    public function processMarkdown($value)
    {
        return strtoupper($value);
    }
}
```

Dans le code Twig, il suffit de changer le nom du filtre appelé pour que le code soit pris en compte :
`{{ article.content|cached_markdown }}`.

Notons qu'on ajoute au constructeur de TwigFilter le paramètre `['is_safe' => ['html']]`, qui permet de vérifier que le HTML 
est "bien formé" (terme à clarifier ici à l'occasion).

Pour à présent appliquer le `MarkdownHelper` dans notre filtre, on va appliquer une **injection de dépendance**, ce qui veut 
juste dire qu'on va faire passer le service via le constructeur de notre `AppExtension`.  
Ce qui donne le code suivant :

```PHP
class AppExtension extends AbstractExtension
{
    private $helper;

    public function __construct(MarkdownHelper $helper) {

        $this->helper = $helper;
    }

    public function getFilters(): array
    {
        return [
            new TwigFilter('cached_markdown', [$this, 'processMarkdown'], ['is_safe' => ['html']]),
        ];
    }

    public function processMarkdown($value)
    {
        return $this->helper->parse($value);
    }
}
```