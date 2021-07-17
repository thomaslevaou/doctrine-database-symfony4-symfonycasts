# All about Entity Repositories

Les articles listés sur la home page étant toujours codés en dur, on va remédier à cela maintenant.  
On commence par mettre à jour la méthode `homepage()` de `ArticleController`, d'une manière analogue 
à ce qui a déjà pu être fait auparavant: 
```PHP
public function homepage(EntityManagerInterface $em)
{
    $repository = $em->getRepository(Article:class);
    $articles = $repository->findAll();

    return $this->render('article/homepage.html.twig', [
        'articles' => $articles
    ]);
}
```

Et mettre à jour le template Twig dans `homepage.html.twig` :
```HTML

<!-- Supporting Articles -->
{% for article in articles %}
<div class="article-container my-1">
    <a href="{{ path('article_show', {slug: article.slug}) }}">
        <img class="article-img" src="{{ asset('images/asteroid.jpeg') }}">
        <div class="article-title d-inline-block pl-3 align-middle">
            <span>{{ article.title }}</span>
            <br>
            <span class="align-left article-details"><img class="article-author-img rounded-circle" src="{{ asset('images/alien-profile.png') }}"> Mike Ferengi </span>
            <span class="pl-5 article-details float-right">
                {{ article.publishedAt ? article.publishedAt|date('Y-m-d') }}
            </span>
        </div>
    </a>
</div>
{% endfor %}
```

On cherche à présent à afficher les articles les plus récents en premier.  
Pour ce faire, on va modifier l'appel aux articles dans `homepage()` :
```PHP
$articles = $repository->findBy([], ['publishedAt' => 'DESC'])
```

Notons que le premier tableau de `findBy()` permet d'ajouter des clauses comme dans un "where" si besoin.  


Maintenant, on va vouloir ne pas afficher les articles qui n'ont pas été publiés (sans date de publication) sur 
la page d'accueil. Le problème c'est qu'avec `findBy()`, il est impossible de faire un "where publishedAt is NOT NULL". 
Du coup on a créer notre reque à la main. 

Un `dump($repository);die;` dans la fonction `homepage()` permet de se rendre compte que le `$repository` est en réalité
une instance de `src/Repository/ArticleRepository`. Le lien entre `Article` et `ArticleRepository` est défini dans 
l'annotation en haut de la classe `Article`. La méthode `findBy` vient d'une classe parente de `ArticleRepository`.  
Et en fait, si on veut ajouter notre propre méthode de recherche de `Article`, il nous suffit de créer une nouvelle 
méthode dans la classe `ArticleRepository`. C'est ce qu'on va faire au chapitre suivant.