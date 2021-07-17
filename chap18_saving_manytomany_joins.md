# Saving a ManyToMany Relation + Joins

Maintenant qu'on a vu qu'une table de jointure a été créée pour nos tags, on va 
voir comment insérer des données dans ce genre de relation.  
On commence tout d'abord par ajouter la fonction `getRandomReferences` donnée 
par le cours, dans la classe `BaseFixture`.  

Ensuite, comme la classe `ArticleFixtures` va à présent demander des fixtures 
de tags, elle doit implémenter `DependentFixtureInterface` pour attendre que les 
tags soient créées avant de générer ses propres fixtures. Ce qui implique, pour
rappel, de devoir implémenter la méthode `getDependencies` pour dire que la classe
ArticleFixture va être dépendant de la classe TagFixture.  

Pour indiquer lors de la création des Fixtures qu'un article doit avoir 
entre 0 et 5 tags existants, on appelle `getRandomReferences` dans l'appel
à `createMany` de `ArticleFixtures`: 
```PHP
$tags = $this->getRandomReferences(Tag::class, $this->faker->numberBetween(0,5));
```

Notons que si on veut vérifier des trucs, on peut faire un `dump` dans la fixture,
et le résultat du dump sera visible en console dans
`./bin/console doctrine:fixtures:load`. On peut alors voir que parfois, les classes
entités ont "enveloppées" d'une classe Proxies, qui n'est là que pour des raisons
d'optimisation.
D'ailleurs si on dump les $tags juste après cette ligne, les données seront 
affichées comme étant `null`. En réalité, les données seront remplies dès que
les données des tags seront demandées (par lazy loading).  
Par exemple, si on appelle `getName()` dans une boucle sur les tags avant de dumper,
comme ci-dessous : 
```PHP
/** @var Tag[] $tags */
$tags = $this->getRandomReferences(Tag::class, $this->faker->numberBetween(0,5));
foreach ($tags as $tag) {
    $tag->getName();
    dump($tag);die;
}
```
Alors le load des fixtures va bien montrer les données du premier tag.
Dans la boucle ci-dessus, on va juste remplacer les deux lignes 
par `$article->addTag($tag);`, et reload les fixtures: des données sont bien 
visible dans la table `article_tag` via la commande doctrine:query:sql.  

La table de jointure est 100% gérée par Doctrine.  

Pour afficher ces tags sur l'interface, on se content de les afficher en Twig
dans `article/show.html.twig` et dans `homepage.html.twig`, via une boucle for 
assez basique :
```HTML
{% for tag in article.tags %}
    <span class="badge badge-secondary">{{ tag.name }}</span>
{% endfor %}
```

Le problème, c'est qu'on est passé de 8 à 15 requêtes SQL... Pour chacun 
des articles, une requête a été générée pour trouver les tags correspondants.  
On se retrouve alors avec un nouveau problème du N+1, qu'on va résoudre dans 
le chapitre suivant.