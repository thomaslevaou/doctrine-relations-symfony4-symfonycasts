# Adding the ManyToOne Relation

Pour qu'un article ait plusieurs commentaires, chaque commentaire devra avoir 
son id d'article associé. 
Donc pour modifier notre Comment, on va donc refaire `./bin/console make:entity` en 
entrant Comment comme nom de classe.  
Mais au moment d'entrer notre nouveau nom de propriété, on va entrer `article` avec
cette classe. En type, on va entrer "relation". De ce fait, la commande va nous 
demander la classe qui devra être associée à cette entité.  

La commande nous propose alors une des 4 relations possibles proposées par Doctrine:
ManyToOne, OneToMany, ManyToMany, et OneToOne.  

La relation ManyToOne, qu'on va appliquer ici, est la relation la plus fréquemment
utilisée en Doctrine.

Une fois qu'on l'a entrée, on doit préciser si la propriété peut être nulle (est-ce 
qu'on peut avoir un commentaire sans article ici quoi). Ici, on va donc mettre "no".  

La commande nous demande ensuite si on veut que la classe `Article` ait une nouvelle
propriété pour qu'on puisse ajouter ou modifier les commentaires depuis celui-ci
(par exemple si on veut un `$article->getComments()`). Parce que jusqu'ici dans tous
les cas on pourra accéder à l'article depuis un commentaire, mais on ne sait pas 
encore si un article pourra accéder à tous ses commentaires dans l'état actuel 
du code PHP. Comme on va s'en servir ici, on va mettre `yes` (la valeur par défaut).

La propriété dans la classe `Article` aura le nom par défaut (`comments`).  
Ensuite, la commande nous demande si on veut avoir des *Orphan removals**
(suppressions orphelines) dans les commentaires de l'article.  
J'ai l'impression que cette commande re-demande si on peut avoir des commentaires
sans article (question à laquelle on a déjà répondu "non" tout à l'heure ?).  
Ici, on va donc re-répondre "no". Apparemment il y a des cas un peu touchy 
où on devra mettre "yes" sur certaines formulaires, mais on verra ça plus tard
si besoin. En tout cas maintenant, on peut appuyer sur la touche entrée pour dire
qu'on a fini d'ajouter notre relation.

On peut bien voir dans notre entité `Comment` que la relation à l'`Article` a bien 
été ajoutée :
```PHP
/**
 * @ORM\ManyToOne(targetEntity=Article::class, inversedBy="comments")
 * @ORM\JoinColumn(nullable=false)
 */
private $article;
```

Dans la classe `Article`, on peut également y voir les nouveaux codes ajoutés suite
à nos indications de relations :
```PHP

    /**
     * @ORM\OneToMany(targetEntity=Comment::class, mappedBy="article")
     */
    private $comments;

    public function __construct()
    {
        $this->comments = new ArrayCollection();
    }
    ...
    
    /**
     * @return Collection|Comment[]
     */
    public function getComments(): Collection
    {
        return $this->comments;
    }

    public function addComment(Comment $comment): self
    {
        if (!$this->comments->contains($comment)) {
            $this->comments[] = $comment;
            $comment->setArticle($this);
        }

        return $this;
    }

    public function removeComment(Comment $comment): self
    {
        if ($this->comments->removeElement($comment)) {
            // set the owning side to null (unless already changed)
            if ($comment->getArticle() === $this) {
                $comment->setArticle(null);
            }
        }

        return $this;
    }
```

Ici, on dit que notre `Article` est reliée à une **Collection** de commentaires.
Quand ce genre de relation à une collection arrive, le constructeur de la classe
ayant une collection (`Article` ici), est automatiquement mis à jour pour inclure une
instanciation de la classe `ArrayCollection` dans sa propriété concernée (`comments`
ici).  
Je pense qu'il s'agit d'une collection similaire à celles du Java. En pratique, on se 
servira de cette collection comme d'un tableau normal. Le fait qu'on utilise une 
collection et pas un tableau est uniquement pour des "raisons internes à Doctrine".  

Comme on peut voir dans la classe `Article`, les relations `OneToMany` et `ManyToOne`
décrivent la même relation, mais dans les sens opposés.  

Puis rebelote:
```
./bin/console make:migration
Vérifier les bonnes modifs dans le fichier de migrations
./bin/console doctrine:migrations:migrate
```