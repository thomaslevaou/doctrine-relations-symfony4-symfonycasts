# Fixtures References & Relating Objects

Au lieu de balancer toutes nos fixtures dans `ArticleFixtures`, on va créer une fixture
pour nos commentaires: `CommentFixture`, pour rappel toujours via la commande 
`./bin/console make:fixture`.  

Comme précédemment, on peut appeler dans `loadData()` la fonction `createMany()` et 
générer des données de commentaire, comme sur le code suivant :
```PHP
class CommentFixture extends BaseFixture
{
    protected function loadData(ObjectManager $manager)
    {
        $this->createMany(Comment::class, 100, function(Comment $comment) {
            $comment->setContent(
                $this->faker->boolean ? $this->faker->paragraph : $this->faker->sentences(2, true)
            );

            $comment->setAuthorName($this->faker->name);
            $comment->setCreatedAt($this->faker->dateTimeBetween('-1 months', '-1 seconds'));

            $comment->setArticle($this->getReference(Article::class.'_0'));
        });
        $manager->flush();
    }
}
```

Notons que la commande `$comment->setArticle($this->getReference(Article::class.'_0'));`
permet d'associer le commentaire à l'article. En gros on prend une référence 
qu'on a créée via la méthode `addReference()` utilisée dans `BaseFixture` (qui a
l'air d'agir comme un système de pseudo-pointeurs C++) et on associe cet article 
au commentaire qu'on a créé. 

Du coup on a 100 commentaires associés au même article comme ça.