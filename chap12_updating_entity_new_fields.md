# Updating an Entity with New Fields

Le but de ce chapitre est d'ajouter trois nouveaux champs dans la table `Article` (L'image de l'article, l'auteur
et le nombre de likes) pour que les détails d'un article ne soient plus écrits en dur.  

Une solution pour ajouter un champ dans l'entité est de le faire "à la main" dans la classe `Article` : ajouter un nouvel
attribut, et ses accesseurs en lecture et écriture associés.  
Mais le moyen le plus efficace est de passer par la commande `php bin/console make entity` (la même que celle utilisée 
dans le chapitre 2). Sauf que là, on va renseigner une classe qui existe déjà (Article), et la commande comprend directement 
qu'on souhaite ajouter de nouveaux champs à cette entité. 
Ainsi, on va pouvoir ajouter les nouveaux champs `author` (qui sera une string pour le moment, en attendant de créer 
l'entité Author et de faire le lien entre les deux), `heartCount` (un entier) et `imageFilename` (string, qui va correspondre
aux paths de différentes images déjà disponibles dans `public/images/`).  

On peut alors voir que les nouveaux attributs et accesseurs ont bien été ajoutés dans la classe `Article`, ce qui 
permet de rendre possible l'exécution de `./bin/console make:migration`.  
On peut alors vérifier que les requêtes exécutées dans le dernier fichier du dossier `migrations` sont bien celles auxquelles 
nous nous attendions. Puis, comme précédemment, on peut faire `./bin/console doctrine:migration:migrate` pour appliquer 
la migration en bdd. 

La valeur de `nullable` dans une Entité valant par défaut "false", si on ne voit pas de `nullable=true` dans un champ, alors 
sa valeur vaut par défaut false et donc le champ est obligatoire.  

Pour donner une valeur par défaut à un champ d'une entité, il suffit de mettre une valeur par défaut dans le code PHP de l'attribut :
```PHP
/**
 * @ORM\Column(type="integer")
 */
private $heartCount = 0;
```


Dans la méthode `new()` de `ArticleAdminController`, on peut alors mettre à jour les champs de l'entité, pour l'instant en dur :
```PHP
$article->setAuthor('Mike Ferengi')->setHeartCount(rand(5,100))->setImageFilename('asteroids.jpeg');
```

On va à présent supprimer toutes les données en base, ce qui peut être fait via la commande
`./bin/console doctrine:query:sql "TRUNCATE TABLE article"`
On peut alors voir sur la page d'accueil qu'il n'y a plus aucun article dans la liste. 
Les articles sont recréés comme auparavant en entrant l'adresse `admin/article/new` dans le navigateur, à plusieurs reprises. 

On change ensuite dans `homepage.html.twig` et `show.html.twig` les appels à `author` et `heartCount` par des appels de
l'entité au lieu des données en dur, comme on a déjà fait précédemment. Concernant, le path de l'image, on pourrait utiliser
l'opérateur de concaténation de Twig `~`, mais la meilleure option est de créer d'abord une nouvelle méthode dans l'entité 
Article avec le code suivant : 
```PHP
public function getImagePath()
{
    return 'images/'.$this->getImageFilename();
}
```
Rappelons que le "/" en début de chaîne de caractères ici n'est pas obligatoire, Symfony le rajoute automatiquement.

Ce qui permet d'appeler directement le `imagePath` dans le code Twig :
```HTML
<img class="article-img" src="{{ asset(article.imagePath) }}">
```