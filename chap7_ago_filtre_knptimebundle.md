# ago Filter with KnpTimeBundle

Jusqu'à présent le "4 hours ago" censé indiquer la date de publication de l'article
a toujours été écrit en dur. On va à présent y remédier. 

On commence par remplacer le code en dur dans la vue Twig :
```HTML
<span class="pl-2 article-details">
    {{ article.publishedAt ? article.publishedAt|date('Y-m-d') : 'unpublished' }}
</span>
```

Ce qui permet de voir que, pour un article daté comme celui sur
http://localhost:8000/news/why-asteroids-taste-like-bacon-775, une date de publication
en format YYY-MM-dd sur la page.
Notons que certains filtres, comme "date" ici, peuvent avoir des paramètres.  

Pour avoir un filtre qui va plutôt afficher la date dans un style comme "il y a X temps", 
on va installer le **KnpTimeBundle** via la commande suivante :
`composer require knplabs/knp-time-bundle`

On peut y voir qu'une Recipe a crée un dossier `translations/`, pour pouvoir gérer les 
traductions si on en a un jour besoin.
Il suffit alors de remplacer le filtre de la date par un `ago` :

```HTML
<span class="pl-2 article-details">
    {{ article.publishedAt ? article.publishedAt|ago : 'unpublished' }}
</span>
```

Et on peut voir l'affichage de la date modifié comme souhaité.  

Par contre on a un problème de compatibilité de version entre ce projet Symfony et Slack, qui empêche l'ajout de cette 
extension. Je vais faire sans pour le moment, et me repencherai sur le souci s'il est embêtant à l'avenir. 