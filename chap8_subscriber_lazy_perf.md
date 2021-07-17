# Service Subscriber: Lazy Performance

Normalement en Symfony, un Service n'est pas instancié tant qu'on ne l'utilise pas, pour des raisons de performance.  

Mais si une page qui retourne un template Twig est appelée, alors une instance de la classe `AppExtension` sera générée 
même si on n'utilise aucune de ses fonctions ou filtres. En effet l'instanciation de AppExtension est obligatoire pour que 
que Twig puisse savoir quels sont les filtres/fonctions applicables ou non.  
Et en raison de notre code de `AppExtension`, une instance de `MarkdownHelper` est créée à chaque requête appelant 
une vue Twig. Le jour où on aurait besoin de 36 services dans `AppExtension`, ça va être la misère...

Pour éviter d'instancier tous les Services de `AppExtension` à chaque affichage d'une vue Twig, on va utiliser un 
**Service Subscriber**. Le code de notre `AppExtension` va alors devenir le suivant :
```PHP
class AppExtension extends AbstractExtension implements ServiceSubscriberInterface
{
    private $container;
    
    public function __construct(ContainerInterface $container) {
        $this->container = $container;
    }

    public function getFilters(): array
    {
        return [
            new TwigFilter('cached_markdown', [$this, 'processMarkdown'], ['is_safe' => ['html']]),
        ];
    }

    public function processMarkdown($value)
    {
        return $this->container->get(MarkdownHelper::class)->parse($value);
//        return $this->container->get('foo')->parse($value);
    }

    public static function getSubscribedServices()
    {
        return [
            MarkdownHelper::class
            // 'foo' => MarkdownHelper::class
        ];
    }
}
```

Ce code veut concrètement dire qu'il va créer une sorte de "mini-container" où les services associés sont listés 
dans `getSubscribedServices`, et seront instanciés uniquement lors de l'appel à la méthode `get()`.

On peut vérifier le bon fonctionnement en ajoutant un `die;` en première ligne du constructeur de `MarkdownHelper`:
on peut alors voir que la page de l'article n'est pas affichée, tandis que la page d'accueil affichant tous les articles
(et n'utilisant pas de Markdown) est affichée.

Pour éviter d'ajouter trop de code inutilement, on n'utilisera le système de Service Subscriber que si nécessaire: 
pour les Twig Extensions, les EventSubscribers et les Security Voters.


## Mise à jour de Twig 1.26

Depuis Twig 1.26, une manière plus efficace pour implémenter des Twig "lazy-loaded" consiste à créer une classe `AppRunTime`
implémentant `RuntimeExtensionInterface` dans le dossier `Twig`, et à déplacer certaines parties du code de notre `AppExtension`
dans celle-ci. Les codes deviennent alors les suivants: 

```PHP
class AppExtension extends AbstractExtension
{
    public function getFilters(): array
    {
        return [
            new TwigFilter('cached_markdown', [AppRunTime::class, 'processMarkdown'], ['is_safe' => ['html']]),
        ];
    }
}
```

```PHP
class AppRunTime implements RuntimeExtensionInterface
{
    private $markdownHelper;

    public function __construct(MarkdownHelper $markdownHelper)
    {
        $this->markdownHelper = $markdownHelper;
    }

    public function processMarkdown($value)
    {
        return $this->markdownHelper->parse($value);
    }
}
```