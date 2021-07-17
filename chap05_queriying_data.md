# Querying for Data ! 

Maintenant qu'on a réussi à enregistrer des articles en base, on souhaite les prélever pour les afficher sur notre 
site, à la place des articles affichés en dur jusqu'à présent.  

Pour ce faire, on va appeler le **Repository** associé à l'article, dans la méthode `show()` de `ArticleController`.
On le stocke dans une variable `$repository`:  
```PHP
$repository = $em->getRepository(Article::class);
```

Cette variable `$repository` dispose de plusieurs méthodes déjà définies : `count()`, `findAll()`, `findBy()`, `findOneBy()`, etc.  
Les noms sont assez explicites sur l'utilité de ces méthodes. Mais si besoin de détails sur les arguments à passer, la doc existe.  

Ici, on va récupérer un article de la base de données dont le slug correspond à celui entré par l'utilisateur. Ce qui nous
amène à entrer la commande suivante (L'annotation est juste là pour des raisons de perf) :
```PHP
/** @var Article $article */
$article = $repository->findOneBy(['slug' => $slug]);
```

Le retour de la méthode de `$repository` est un objet instance de la classe `Article`, pas juste un tableau.  
Un peu comme m'expliquait Grégory à mon arrivée à Label, on fait un peu une sorte de DAO.  

Si aucun article avec le slug associé n'est trouvé, une erreur 404 sera retournée :
```PHP
if (!$article) {
    throw $this->createNotFoundException(sprintf('No article for slug "%s"', $slug));
}
```

Si on fait `Ctrl + Click` comme sur la méthode `createNotFoundException()` ici, on accède à la définition de la méthode 
(déjà vu à Label). On peut voir que cette méthode retourne une erreur 404 au navigateur.  
Le message qu'on met en paramètre de cette méthode peut être aussi technique qu'on souhaite: il n'est accessible qu'au développeur 
(c'est un message visible dans le Profiler uniquement). On peut utiliser `createNotFoundException()` pour customiser notre 
page d'erreur 404 comme on le souhaite.  
Customiser sa page 404, 403, 500 ou whatever n'est pas décrit dans ce cours, mais c'est rapide à faire pour peu qu'on cherche un peu 
"Symfony customize error pages" sur le Net.

Lorsqu'on affiche la page avec un slug valide (par exemple ici, `http://localhost:8000/news/why-asteroids-taste-like-bacon-499`
après avoir créé l'article via `http://localhost:8000/admin/article/new`), on peut passer l'instance de article dans le template, 
au lieu de renvoyer trois balises `title`, `slug` et `articleContent` de manière séparée.  
Dans le template Twig, on peut indiquer les attributes de notre nouvel objet en séparant l'instance et l'attribut avec un point :
par exemple, on appelle title en faisant `{{ article.title }}`. Précisons quand même que l'attribut "title" n'est pas public, 
mais si Twig n'a pas accès directement à l'attribut d'un objet, il sait appeler implicitement sont accesseur en lecture s'il existe.

Notons au passage que le markdown peut être lu en twig via le filtre `|markdown`, par exemple ici avec `{{ article.content|markdown }}`.  

Lorsque la page est affichée dans le navigateur, en dev le Profiler dispose d'une option permettant de voir le nombre de requêtes 
SQL exécutées, et le temps prises par chacune. Cette option permet également de voir quelles ont été précisément les requêtes SQL 
exécutées.