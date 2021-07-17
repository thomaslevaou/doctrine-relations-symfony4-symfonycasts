# Saving Relations

Maintenant qu'on a créé notre relation entre Comment et Article, on va voir comment
prendre des commentaires de la base de données et les associer à notre article.

Dans notre fonction d'ajout d'article dans `ArticleFixtures`, on va ajouter des 
instances de la classe `Comment`, de la manière suivante : 
```PHP
$comment1 = new Comment();
$comment1->setAuthorName('Mike Ferengi');
$comment1->setContent('I ate a normal rock once. It did NOT taste like bacon!');
$manager->persist($comment1);
```

En ajoutant un `use ($manager)` lors de la déclaration de la fonction anonyme. 

Seulement, ce code ne suffit pas : si on exécute maintenant 
`./bin/console doctrine:fixtures:load`, la console nous retourne une erreur étant 
donné que article_id ne peut être nul. 

En fait pour associer l'article au commentaire, il suffit d'appeler 
`$comment->setArticle($article)` dans le code ci-dessus, avant l'appel à `persist`.  

Et du coup c'est tout : Doctrine se charge du reste.

Une fois cet oubli corrigé (et un `$comment2` ajouté de la même manière dans la 
foulée), la commande `./bin/console doctrine:fixtures:load` tourne correctement,
et on peut vérifier la présence des commentaires insérés en base par un
`./bin/console doctrine:query:sql 'SELECT * from comment'`.