# Awesome Random Fixtures

Comme on a créé 10 articles (cf `createMany` de `ArticleFixtures`), on peut récupérer
les références des articles (au lieu du 0 en dur) en remplaçant un peu la fin 
du `getReference` dans `CommentFixture`: 
```PHP
$comment->setArticle($this->getReference(Article::class.'_'.$this->faker->numberBetween(0, 9)));
```

Après un `php bin/console doctrine:fixtures:load`, on peut voir avec un 
`php bin/console doctrine:query:sql 'SELECT * FROM comment'` que les articles sont 
créés avec des id différents.

Mais une autre solution plus sympa existe.  

Dans `BaseFixture`, on va ajouter une méthode générique pour obtenir une référence
de n'importe quelle classe, dont le code va être le suivant :
```PHP
protected function getRandomReference(string $className) {
        if (!isset($this->referencesIndex[$className])) {
            $this->referencesIndex[$className] = [];
            foreach ($this->referenceRepository->getReferences() as $key => $ref) {
                if (strpos($key, $className.'_') === 0) {
                    $this->referencesIndex[$className][] = $key;
                }
            }
        }
        if (empty($this->referencesIndex[$className])) {
            throw new \Exception(sprintf('Cannot find any references for class "%s"', $className));
        }
        $randomReferenceKey = $this->faker->randomElement($this->referencesIndex[$className]);
        return $this->getReference($randomReferenceKey);
}
```

Mais attention, ce code ne fonctionne que si la classe `ArticleFixtures` est mise en 
base avant la classe `CommentFixture`. Comme les classes son chargées par ordre 
alphabétique, si on renomme cette classe en `A0CommentFixture`, alors le code ne 
marchera pas.  

De ce fait, les classes de fixtures qui ont besoin du chargement préalable d'une 
autre fixture doivent implémenter l'interface `DependentFixtureInterface`.  
Lors de son implémentation, la méthode `getDependencies` doit être définie. 
On va lui attribuer le code suivant : 
```PHP
public function getDependencies()
{
    return [
        ArticleFixtures::class
    ];
}
```

Ce qui fait alors marcher `php bin/console doctrine:fixtures:load` même lorsque 
notre fixture de commentaires s'appelle `A0CommentFixture`.