# Custom Queries

Il est possible d'écrire directement les requêtes en SQL avec Doctrine. Mais en général, on ne va pas faire ça.  
En effet, Doctrine est une bibliothèque qui peut fonctionner avec beaucoup de SGBD différents, et utilise pour cela
son propre langage de requête, appelé **DQL** (Doctrine Query Language).  

La principale différence réside dans le fait qu'au lieu de directement faire appel aux tables et colonnes de la bdd,
on va utiliser des classes et des propriétés.  
Le but de Doctrine est surtout de faire croire qu'il n'y a pas de base de données derrière.  

Pour créer notre requête à la main, on peut créer une chaîne de caractères DQL directement, mais ce qu'on fera
en général sera d'utiliser le **QueryBuilder**, comme on a déjà vu à Label Emmaüs.  

On peut y voir déjà un exemple proposé dans `ArticleRepository` :
```PHP
return $this->createQueryBuilder('a')
            ->andWhere('a.exampleField = :val')
            ->setParameter('val', $value)
            ->orderBy('a.id', 'ASC')
            ->setMaxResults(10)
            ->getQuery()
            ->getResult()
        ;
```
Notons que les paramètres peuvent être mis dans n'importe quel ordre. De plus, même s'il existe une méthode `where()`,
il vaut mieux toujours utiliser `andWhere()` à la place. En effet `where()` va supprimer tous les `andWhere()` ajoutés 
précédemment, parfois c'est relou.  
DQL et le query builder utilisent les **prepared statements**, ce qui correspond aux ":" pour définir la valeur d'un paramètre
avec la méthode `setParameter()` ensuite, comme déjà vu à Label. Ce qui permet d'éviter des injections SQL.  

Notons que le "a" est juste l'équivalent d'un `SELECT a.* FROM ARTICLE AS a`. Cette lettre pourrait être remplacée par 
n'importe quelle autre chaîne de caractères, tant que les commandes qui y font référence sont mises à jour également.  

Dans notre méthode `findAllPublishedOrderedByNewest()` de `ArticleRepository`, le code va donc être le suivant :
```PHP
/**
 * @return Article[]
 */
public function findAllPublishedOrderedByNewest()
{
    return $this->createQueryBuilder('a')
        ->andWhere('a.publishedAt IS NOT NULL')
        ->orderBy('a.publishedAt', 'DESC')
        ->getQuery()
        ->getResult()
    ;
}
```

La méthode `getQuery()` doit être appelée une fois que la requête est finie, et pour avoir le tableau d'Articles 
du résultat, on doit appeler `getResult()`.  

Pour n'obtenir qu'un seul résultat, on peut s'inspirer du code de la méthode en-dessous `findOneBySomeField()` si besoin.  
La seule différence réside dans le fait qu'au lieu d'appeler `getResult()` à la fin, on appellera `getOneOrNullResult()`.  

On peut alors ajouter notre nouvelle méthode dans le code de `ArticleController`: 
```PHP
$articles = $repository->findAllPublishedOrderedByNewest();
```

Ce qui permet d'afficher uniquement les articles avec une date de publication sur la page d'accueil des articles.  

On peut même optimiser notre code de `homepage()` en appelant directement l'`ArticleRepository` en paramètre :
```PHP
public function homepage(ArticleRepository $repository)
{
    $articles = $repository->findAllPublishedOrderedByNewest();
    return $this->render('article/homepage.html.twig', [
        'articles' => $articles
    ]);
}
```
Ceci fonctionne car tous les repositories sont enregistrés en tant que Service dans le container, donc on peut appliquer 
l'auto-wiring dessus.  

Plus de détails sur les custom queries en Doctrine sont disponibles ici si besoin un jour : https://symfonycasts.com/screencast/doctrine-queries

(C'est en Symfony 2/3 mais le système de Doctrine n'a pas fondamentalement changé depuis).