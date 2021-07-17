# Using Faker for Seeding Data

Les 10 articles générés dans le chapitre précédent sont tous identiques.  
Pour avoir des données plus réalistes à un environnement de production, on peut utiliser une 
bibliothèque appelée **Faker**.  

On l'installe via la commande `composer require fzaninotto/faker --dev` (encore une fois, ce n'est à utiliser qu'en dev !).  

On peut alors indiquer la présence de notre faker dans la classe `BaseFixture`:
```PHP
abstract class BaseFixture extends Fixture
{
    /** @var ObjectManager */
    private $manager;

    /** @var Generator */
    protected $faker;

    abstract protected function loadData(ObjectManager $em);

    public function load(ObjectManager $manager)
    {
        $this->manager = $manager;
        $this->faker = Factory::create();
        $this->loadData($manager);
    }
    ...
}
```

(On sait que `$faker` est un `Generator` après avoir jeté un oeil au code de `Factory::create()`)

Dans `loadData()` de `ArticleFixtures`, on peut alors appliquer quelques méthodes de Faker, 
comme `$this->faker->boolean(70)` qui retourne true dans 70% des appels, ou des fonctions comme 
̀`$this->faker->dateTimeBetween('-100 days', '-1 days')` ou `$this->faker->numberBetween(5, 100)` dont 
les noms parlent d'eux-mêmes. 
Pour générer des titres d'articles, auteurs et photos aléatoirement, on va faire appel à la méthode 
`$this->faker->randomElement` parmi des tableaux statiques en attributs privés. 

Du coup concrètement, notre code de `ArticleFixtures` ressemble au final à ceci: 
```PHP
class ArticleFixtures extends BaseFixture
{

    private static $articleTitles = [
        'Why Asteroids Taste Like Bacon',
        'Life on Planet Mercury: Tan, Relaxing and Fabulous',
        'Light Speed Travel: Fountain of Youth or Fallacy',
    ];
    private static $articleImages = [
        'asteroid.jpeg',
        'mercury.jpeg',
        'lightspeed.png',
    ];
    private static $articleAuthors = [
        'Mike Ferengi',
        'Amy Oort',
    ];

    protected function loadData(ObjectManager $manager)
    {
        $this->createMany(Article::class, 10, function(Article $article, $count) {
            $article->setTitle($this->faker->randomElement(self::$articleTitles))
                ->setSlug($this->faker->slug)
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


            if ($this->faker->boolean(70)) {
                $article->setPublishedAt($this->faker->dateTimeBetween('-100 days', '-1 days'));
            }

            $article->setAuthor($this->faker->randomElement(self::$articleAuthors))
                ->setHeartCount($this->faker->numberBetween(5, 100))
                ->setImageFilename($this->faker->randomElement(self::$articleImages));
        });
        $manager->flush();
    }
}
```

Et après avoir regénéré la base de données, toujours via un `php bin/console doctrine:fixtures:load`, on a des données 
réalistes.