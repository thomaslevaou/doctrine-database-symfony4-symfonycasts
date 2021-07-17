# Query Logic Re-use & Shortcuts

Avec le QueryBuilder, certaines logiques sont réutilisables.  

Pour réutiliser certaines parties, on va isoler la partie qui va être utilisée entre deux requêtes custom.  

Ce qui donne le résultat suivant dans `ArticleRepository` :
```PHP
class ArticleRepository extends ServiceEntityRepository
{
    public function __construct(RegistryInterface $registry)
    {
        parent::__construct($registry, Article::class);
    }

    /**
     * @return Article[]
     */
    public function findAllPublishedOrderedByNewest(): array
    {
        return $this->addIsPublishedQueryBuilder()
            ->orderBy('a.publishedAt', 'DESC')
            ->getQuery()
            ->getResult()
        ;
    }

    private function addIsPublishedQueryBuilder(QueryBuilder $qb = null)
    {
        return $this->getOrCreateQueryBuilder($qb)
            ->andWhere('a.publishedAt IS NOT NULL');
    }

    private function getOrCreateQueryBuilder(QueryBuilder $qb = null)
    {
        return $qb ?: $this->createQueryBuilder('a');
    }
}
```

Le signe `?:` est juste là pour dire que si le QueryBuilder `$qb` existe, il est retourné, sinon, la condition à droite 
du `?:` s'applique.  


Dans la méthode `show()` de `ArticleController`, notre traitement fait à la main pour trouver un article et afficher une 
erreur 404 s'il n'est pas trouvé peut être géré automatiquement par Symfony. Autrement dit, le code ci-dessous 
fonctionne sans problème :
```PHP
    /**
     * @Route("/news/{slug}", name="article_show")
     */
    public function show(Article $article, SlackClient $slack)
    {
        if ($article->getSlug() === 'khaaaaaan') {
            $slack->sendMessage('Kahn', 'Ah, Kirk, my old friend...');
        }
        
        $comments = [
            'I ate a normal rock once. It  did NOT taste like bacon!',
            'Woohoo! I\'m going on an all-asteroid diet!',
            'I like bacon too! Buy some from my site! bakinsomebacon.com',
        ];
        
        return $this->render('article/show.html.twig', [
            'article' => $article,
            'comments' => $comments,
        ]);
    }
```
En fait, si on passe à un contrôleur une entité, Symfony va automatiquement générer certaines requêtes 
à partir des paramètres de la route. Ca veut dire que si on a un paramètre `slug` dans notre Route, et que 
nos `Article` ont un attribut `slug`, alors Symfony saura automatiquement trouver l'article dont le slug 
correspond au slug de la route. En fait Symfony va appliquer le même code que celui qu'on a écrit à la main
auparavant, à savoir trouver l'article s'il existe, sinon retourner une erreur 404. On appelle cette fonctionnalité
le `ParamConverter`.  

Du coup maintenant si on a besoin de faire des requêtes simples, on peut laisser Symfony gérer tout seul. Et pour 
les requêtes un peu plus custom, il suffit d'auto-wirer notre `ArticleRepository`.