# Updating an Entity

On rappelle que dans un chapitre du premier cours, le clic sur le coeur pour liker un article appelait 
en AJAX la méthode `toggleArticleHeart()` de `ArticleController`, qui renvoyait un nombre aléatoire entre 
1 et 100 pour simuler un nouveau nombre de likes.  

Comme vu dans le chapitre précédent, on peut remplacer le `slug` directement par l'entité dans `toggleArticleHeart`, 
et Symfony trouvera automatiquement un article avec un slug correspondant au paramètre. Pour mettre à jour l'article, 
il suffit de remplacer le code existant dans la méthode par le code suivant :
```PHP
/**
 * @Route("/news/{slug}/heart", name="article_toggle_heart", methods={"POST"})
 */
public function toggleArticleHeart(Article $article, LoggerInterface $logger, EntityManagerInterface $em)
{
    $article->setHeartCount($article->getHeartCount() + 1);
    $em->flush();
    $logger->info('Article is being hearted!');

    return new JsonResponse(['hearts' => $article->getHeartCount()]);
}
```

Notons que la ligne `$article->setHeartCount($article->getHeartCount() + 1);` risque de poser un problème de  
_race_ condition si 10 utilisateurs likent l'article en même temps, mais ce n'est pas le sujet de ce cours. Il sera 
intéressant de se renseigner dessus si le problème se produit de mon côté un jour.

Notons aussi que la commande `$em->persist()` n'est pas utile pour une mise à jour : la commande `$em->flush()` ici suffit. 

On peut alors voir que le nombre d'articles est mis à jour lors du clic sur le like.  
Mais il reste à limiter le nombre de likes à 1 max par utilisateur, ce qu'on pourra faire quand on verra les parties sur les utilisateurs
et la sécurité.  

En général en Symfony, on évite d'écrire la logique directement dans le contrôleur. On va plutôt créer un Service
en dehors de la méthode, et y mettre la logique dedans.  
Ce qui se traduit par l'ajout d'une nouvelle méthode dans `Article` :
```PHP
public function incrementHeartCount(): self
{
    $this->heartCount = $this->heartCount + 1;

    return $this;
}
```
Et son appel dans `toggleHearCount` : 
```PHP
/**
 * @Route("/news/{slug}/heart", name="article_toggle_heart", methods={"POST"})
 */
public function toggleArticleHeart(Article $article, LoggerInterface $logger, EntityManagerInterface $em)
{
    $article->incrementHeartCount();
    $em->flush();

    $logger->info('Article is being hearted!');

    return new JsonResponse(['hearts' => $article->getHeartCount()]);
}
```


Notons que si des accesseurs dans les entités sont inutiles, on peut les supprimer pour clarifier le code. 
A condition bien sûr de bien savoir à quoi sert chaque accesseur en écriture/lecture et d'être sûr qu'on peut les virer
dans les utilisations futures de l'application.