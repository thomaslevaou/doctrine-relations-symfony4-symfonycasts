# Adding a Comment Entity

~~Pour rappel, pour faire tourner correctement le `composer install` en installant 
le code du cours, on doit :
~~- Supprimer les utilisations de Slack dans le projet, c'est-à-dire dans le fichier
  `src/Service/SlackClient.php`, dans `config/bundles.php`, et le contenu du fichier
  `config/packages/nexy_slack.yaml` doit être supprimé.~~
- ~~Supprimer l'appel au bundle HttplugBundle, et le contenu de
  `config/packages/httplug.yaml`.~~
- ~~Exécuter `composer remove nexylan/slack-bundle`, puis `composer update`
On peut alors exécuter ce `composer install`.~~ 

Un problème issu du modèle déprécié `DoctrineCacheBundle` semble rendre impossible
l'utilisation du code du cours. Donc, pour le moment, on va utiliser le code du cours
précédent.  
Pour les prochaines parties du cours, on évitera autant que possible d'utiliser les
codes proposés par SymfonyCasts, mais à la place d'utiliser le code de la partie
précédente, pour éviter des problèmes de compatibilité.

Bref.

On a vu dans la partie précédente comment créer un article en base.  
Mainenant, on va voir comment créer une entité `Comment` pour représenter 
les commentaires de cet article (qui sont toujours codés en dur actuellement 
dans notre projet fil rouge).  
On rappelle que pour ce faire, on exécute la commande 
`php bin/console make:entity`, en nommant notre nouvelle entité "Comment" et qui
aura pour attributs "authorName" et "content".  
Une fois cela fait, on va aller dans notre nouvelle classe créée dans 
`src/Entity/Comment`, et ajouter un `use TimestampableEntity;` qui pour rappel
ajoute automatiquement les champs "createdAt" et "updatedAt" dans l'netité 
et la bdd.  

Une fois cela fait, on peut générer le fichier de migration : `./bin/console make:migration`.
Et après avoir vérifié qu'il est ok (toujours bon à vérifie quand on travaille sur des projets à plusieurs 
branches), on peut faire un `./bin/console doctrine:migrations:migrate`.
