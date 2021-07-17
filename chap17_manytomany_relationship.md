# ManyToMany Relationship

Pour ajouter notre relation ManyToMany, on va exécuter `./bin/console make:entity`
comme d'habitude. On va commencer par mettre à jour `Article` ici.  
Le nom de la nouvelle propriété va être `tags`, et son type sera `relation` 
(pour lancer l'assistant de relation). La relation sera liée à l'entité `Tag`,
le type de relation `ManyToMany`. A la question nous demandant si on veut que 
le tag puisse aussi accéder aux articles, on va mettre `yes`. Cette propriété 
dans la classe Tag sera appelée `articles`.  

La relation `ManyToMany` a bien été crée dans l'entité Article. Les tags sont bien 
initialisés dans le constructeur en tant qu'`ArrayCollection`.  
La relation a aussi été créée dans la classe `Tag`.  

La relation dans Article dispose de l'annotation suivante :
```PHP
/**
 * @ORM\ManyToMany(targetEntity=Tag::class, inversedBy="articles")
 */
private $tags;
```

Tandis que celle dans Tags est celle-ci :
```PHP
/**
 * @ORM\ManyToMany(targetEntity=Article::class, mappedBy="tags")
 */
private $articles;
```

Comme Doctrine regarde en premier l'owning side pour créer les données dans la base de données,
un des deux côtés de la relation doit être en owning side, l'autre en inverse side. Comme 
Article est l'entité qu'on a mise à jour dans notre make:entity, alors c'est cette entité qui 
est en owning side. D'où l'annotation `inversedBy="articles"` lors de l'annotation sur les tags
dans la classe `Article`. Et à l'inverse, dans la classe `Tag`, la classe Article est "mappée" 
par les Tags, ce qui veut dire que `Tag` est l'inverse side.
Ca veut concrètement dire que si on ajoute des articles à un tag uniquement, Doctrine ne fera rien.
Mais si on ajoute des tags à un article, Doctrine mettra à jour les données en base. 
D'où le fait que dans les méthodes auto-générées par make:entity, le `addArticle` de 
la classe `Tag` appelle les deux sens de la relation, tandis que `addTag` de `Article` peut 
se permettre de ne mettre à jour que son tableau de Tag sans que ça pose problème.

On peut à présent générer la migration : `./bin/console make:migration`, et voir qu'une nouvelle 
table de jointure va être créée par ce fichier de migration. C'est le seul type de table qui 
n'a pas d'entité associée en Symfony. On peut ensuite faire le classique
`./bin/console doctrine:migrations:migrate`.