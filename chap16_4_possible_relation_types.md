# The 4 (2?) Possible Relation Types

En fait, dire qu'il existe 4 types de relations entre les entités (ManyToOne,
OneToMany, ManyToMany et OneToOne) n'est pas tout à fait vrai.  
On a déjà vu que les relations ManyToOne et OneToMany sont juste deux points de vue
différents pour voir la même relation.

Une relation OneToOne peut être utile quand dans une bdd, on doit relier un profil
à un utilisateur. Mais en base de données, elle sera implémentée d'une manière 
identique à une ManyToOne. La seule différence c'est que le `user_id` de la table
`profile` ici sera paramétré comme `unique` en base (deux profils ne pourront pas 
avoir le même `user_id`). Il n'y aura pas de profile_id dans la table user, c'est
inutile, et potentiellement sujet à de potentielles redondances ou incohérences 
entre les deux côtés. Doctrine saura trouver la relation en sens inverse si besoin.

Et donc, il n'y a que 2 "vrais" types de relation :
les `ManyToOne/OneToMany/OneToOne` d'un côté, et les `ManyToMany` de l'autre.  

Une relation `ManyToMany` peut par exemple être utilisée si on souhaite créer 
des tags pour les articles : un tag pourra être associé à plusieurs articles, et 
un article pourra être associé à plusieurs tags. C'est ce qu'on va mettre en place :
- On crée l'entité via un `./bin/console make:entity` avec deux attributs string :
`name` et `slug`
- En haut de l'entité, on ajoute notre Trait : `use TimestampableEntity;`
- On fait auto-générer le slug avec l'annotation `Gedmo`, comme on fait dans
l'Article au chapitre 16 de la 3ème partie;
- On précise que le slug est unique, via un `unique=true` dans `@ORM\Column`.
- On crée le fichier de migration : `./bin/console make:migration` et on vérifie
  que le fichier créé n'est pas chelou;
- On lance la migration : `./bin/console doctrine:migrations:migrate`
- On crée une classe pour créer des fixtures : `./bin/console make:fixture`, qu'on 
va appeler `TagFixture`, et à laquelle on va associer le code suivant :
```PHP
class TagFixture extends BaseFixture
{
    protected function loadData(ObjectManager $manager)
    {
        $this->createMany(Tag::class, 10, function(Tag $tag){
            $tag->setName($this->faker->realText(20));
        });
        $manager->flush();
    }
}
```
avant de l'appliquer avec `./bin/console doctrine:fixtures:load` (on aurait pu 
utiliser la méthode `word` de faker au lieu de `realText`, mais cette dernière 
méthode donnera des résultats un peu plus réalistes).

On va alors pouvoir créer la relation ManyToMany dans le chapitre suivant.
