# Creating an Entity Class

Doctrine est un **ORM**, c'est-à-dire un Object Relational Mapper.  
Concrètement, et comme on a déjà vu à Label, ça veut dire que chaque table de la base de données aura sa classe associée 
dans le code.  

Doctrine appelle les classes qui sont sauvegardées en base de données des **Entités**.   

On peut ainsi créer une Entité via la commande `./bin/console make:entity` (autorisée via le MakerBundle installé dans
la partie 2 du tuto). Il suffit alors de renseigner le nom de la classe (qui crée deux classes dans `Entity/` et dans `Repository/`), 
ses propriétés, le type de la propriété, (on peut avoir des infos sur les proriétés en entrant `?`, renseigner par ex. string va créer une 
colonne VARCHAR en MySQL), ses infos du type longueur de la chaîne de caractère et son autorisation NULL.  
Pour les gros blocs de texte, on utilisera le type `text` suggéré par le `?` (pas string).  
Une fois qu'on a ajouté toutes nos propriétés, on peut indiquer qu'elle peut être créée en appuyant sur la touche `Entrée`.  

La classe a alors été créée dans `src/Entity`, avec des accesseurs en lecture et écriture pour chacun des attributs.  
Sa classe associée dans `ArticleRepository` a également été créée. 

Ce qui rend l'Entité (dans `src/Entity`) différente des classes PHP habituelle est ses annotations. Elles permettent de savoir
comment stocker en base chacun des champs, un peu comme on a déjà vu à Label. 
Toutes les annotations en Doctrine sont listées sur https://www.doctrine-project.org/projects/doctrine-orm/en/2.8/reference/annotations-reference.html