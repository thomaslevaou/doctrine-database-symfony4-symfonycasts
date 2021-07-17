# Fixtures : Seeding Dummy Data!

Au lieu de créer des données très similaires à chaque appel de `admin/article/new`, on va à la place créer un gros
jeu de données aléatoires. Ce qui facilitera les développements.  
Pour ce faire, on va faire appel à la bibliothèque **DoctrineFixturesBundle**.  

Après avoir supprimé les utilisations de slack dans le projet, fait un `composer remove nexylan/slack-bundle`, puis 
un `composer update`, on peut installer le DoctrineFixturesBundles via la commande `composer require orm-fixtures:3.0.2 --dev`
(La version 3.0.2 pour ce tuto ici). 
Attention à bien préciser le `--dev`, on ne veut surtout pas de fixtures en production !

On peut alors générer notre première classe de fixture grâce à la commande `./bin/console make:fixtures`.  
On va l'appeler `ArticleFixtures`. En général, on aura une classe fixtures par entité ou par groupe d'entités.  
Une fois le nom de la classe entré, un nouveau dossier `DataFixtures` a été créé, contenant notre nouvelle 
classe `ArticleFixtures`.  

Dans la méthode `load()` de `ArticleFixtures`, on va placer à peu de choses près le code de la méthode `new()` qui était
présent dans `ArticleAdminController` :
```PHP
public function load(ObjectManager $manager)
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


        if (rand(1, 10) > 2) {
            $article->setPublishedAt(new \DateTime(sprintf('-%d days', rand(1, 100))));
        }

        $article->setAuthor('Mike Ferengi')->setHeartCount(rand(5,100))->setImageFilename('asteroid.jpeg');

        $manager->persist($article);
        $manager->flush();
}
```

ObjectManager est une interface implémentée par EntityManager. Pour le moment, on peut dire que les deux reviennent au même.
Et pour tester la bonne génération de cette première fixture, on va appliquer la commande `php bin/console doctrine:fixtures:load`.  
Comme cette commande indique, la base de données est purgée avant d'appliquer la fixture (une fois de plus, à ne pas appliquer en production...).

Pour créer plusieurs articles, une première solution est de créer une boucle for pour créer par exemple 10 articles d'un coup.  

Mais à la place, on va procéder d'une manière plus générique : la boucle de création va être placée dans une nouvelle classe
abstraite, qu'on a appelée `BaseFixture`, et dont le code va être le suivant :
```PHP
abstract class BaseFixture extends Fixture
{
    /** @var ObjectManager */
    private $manager;

    abstract protected function loadData(ObjectManager $em);

    public function load(ObjectManager $manager)
    {
        $this->manager = $manager;
        $this->loadData($manager);
    }

    protected function createMany(string $className, int $count, callable $factory)
    {
        for ($i = 0; $i < $count; $i++) { 
            $entity = new $className();
            $factory($entity, $i);
            $this->manager->persist($entity);
            // store for usage later as App\Entity\ClassName_#COUNT#
            $this->addReference($className . '_' . $i, $entity);
        }
    }
}
```

Ainsi, il suffit de faire hériter notre classe `ArticleFixtures` de cette classe, puis d'appeler la fonction `loadData` 
au lieu de `load`, puis d'appeler `createMany` avec les bons paramètres pour pouvoir mettre le tout en place :
```PHP
protected function loadData(ObjectManager $manager)
{
    $this->createMany(Article::class, 10, function(Article $article, $count) {
        $article->setTitle('Why Asteroids Taste Like Bacon')
            ->setSlug('why-asteroids-taste-like-bacon-' . $count)
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
EOF
            );


        if (rand(1, 10) > 2) {
            $article->setPublishedAt(new \DateTime(sprintf('-%d days', rand(1, 100))));
        }

        $article->setAuthor('Mike Ferengi')->setHeartCount(rand(5, 100))->setImageFilename('asteroid.jpeg');
    });
    $manager->flush();
}
```

Et la commande `php bin/console doctrine:fixtures:load` va créer 10 articles au lieu d'un seul.