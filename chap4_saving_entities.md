# Saving Entities

15h35 - 

On va à présent chercher à enregistrer un article dans la base de données.  
On commence par créer le début de la classe PHP qui permettra de faire cela, en instanciant l'Entité créée précédemment :
```PHP
<?php


namespace App\Controller;


use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ArticleAdminController extends AbstractController
{
    /**
     * @Route("/admin/article/new")
     */
    public function new()
    {
        $article = new Article();
        $article->setTitle('Why Asteroids Taste Like Bacon')
            ->setSlug('why-asteroids-taste-like-bacon-'.rand(100,999))
            ->setContent(<<<EOF
Spicy **jalapeno bacon** ipsum dolor amet veniam shank in dolore. Ham hock nisi landjaeger cow,
lorem proident [beef ribs](https://baconipsum.com/) aute enim veniam ut cillum pork chuck picanha. Dolore reprehenderit
labore minim pork belly spare ribs cupim short loin in. Elit exercitation eiusmod dolore cow
**turkey** shank eu pork belly meatball non cupim.

Laboris beef ribs fatback fugiat eiusmod jowl kielbasa alcatra dolore velit ea ball tip. Pariatur
laboris sunt venison, et laborum dolore minim non meatball. Shankle eu flank aliqua shoulder,
capicola biltong frankfurter boudin cupim officia. Exercitation fugiat consectetur ham. Adipisicing
picanha shank et filet mignon pork belly ut ullamco. Irure velit turducken ground round doner incididunt
occaecat lorem meatball prosciutto quis strip steak.

Meatball adipisicing ribeye bacon strip steak eu. Consectetur ham hock pork hamburger enim strip steak
mollit quis officia meatloaf tri-tip swine. Cow ut reprehenderit, buffalo incididunt in filet mignon
strip steak pork belly aliquip capicola officia. Labore deserunt esse chicken lorem shoulder tail consectetur
cow est ribeye adipisicing. Pig hamburger pork belly enim. Do porchetta minim capicola irure pancetta chuck
fugiat.
EOF)
        ;

        //publish most articles
        if (rand(1, 10) > 2) {
            $article->setPublishedAt(new \DateTime(sprintf('-%d days', rand(1, 100))));
        }
        return new Response('space rocks... include comets, asteroids & meteroids');
    }
}
```

Notons que chaque accesseur en écriture de la classe Entity retourne un `$this`, ce qui permet d'enchaîner des modifications 
d'attributs sur une seule ligne : grâce à ça, on peut faire par exemple `$this->setTitle("poulichette")->setSlug("Poulichette2");`.  

Le `DoctrineBundle` ajoute des services, dont un appelé **EntityManager**, avec comme type-hint `EntityManagerInterface`.

Pour enregistrer l'article en base, on ajoute `EntityManagerInterface $em` en paramètre de la méthode `new()`, et on applique
les deux lignes suivantes à la fin de la méthode, avant le `return` :
```PHP
$em->persist($article);
$em->flush();
```

La commande `persist()` est là pour indiquer qu'on souhaite enregistrer l'article, tandis que la commande `flush()` l'insère
en base.  
La séparation en deux commandes est là pour des raisons d'optimisation: on peut vouloir faire 10 `persist()` pour 10 articles,
qui seront insérés d'un coup en base via un unique `flush`.


On peut préciser l'id et le slug de l'article créé dans le retour (on rappelle que l'id est auto-généré dans l'entité):
```PHP
return new Response(sprintf(
            'Hiya! New Article id: #%d slug: %s',
            $article->getId(),
            $article->getSlug()
        ));
```

Ce qui donne un affichage différent à chaque entrée de `http://localhost:8000/admin/article/new` dans le navigateur.  

Un moyen pour vérifier la bonne présence de l'article en base de données (autre que l'utilisation de Adminer ou PhpMyAdmin) 
est d'appliquer la commande `php bin/console doctrine:query:sql "SELECT * FROM article"`, qui retourne les résultats de la requête sous 
forme de tableau PHP. Notons que la table `article` a été créée en minuscules même si l'entité commence avec un A majuscule, 
car Doctrine crée les tables et noms de colonnes en snake case.