# **Introduction**



Dans tous les domaines, il est très utile de maîtriser plusieurs outils qui font, ou pas, la même chose. Mais ce qui est encore plus utile et plus important, c’est de savoir quand et pourquoi choisir un outil plutôt qu’un autre. Ceci est applicable aussi en informatique.

Regardez le tableau suivant qui a été récupéré de la documentation Scala :

|        | head | tail | apply* | append | insert |
| ------ | ---- | ---- | ------ | ------ | ------ |
| List   | C    | C    | L      | L      | -      |
| Stream | C    | C    | L      | L      | -      |
| Vector | eC   | eC   | eC     | eC     | eC     |

**Récupération*

C: Complexité constante.

eC: Complexité effectively constant.

L: Linéaire.

Ce tableau représente les différentes complexités des opérations de base de quelques structures de données dans Scala.

Regardons un peu la troisième ligne qui correspond à Vector. On remarque que toutes les opérations ont une complexité qui est égale à e.C (Ce qui veut dire en anglais effectively constant time).

Dans ce tuto, je vais essayer de vous expliquer le sens et la provenance de cette complexité.





# Hein,Vector ? Qu’est ce que c’est déjà ? Et comment ça marche ?


Commençons, pour ceux qui l’ont oublié, par un rappel sur la définition et l’utilisation de Vector en Scala.

Encore une fois, je vais me référer à la [documentation officielle](https://www.scala-lang.org/api/2.12.2/scala/collection/immutable/Vector.html) afin d’en extraire la définition :  

>  *Vector est une structure de données immuable qui fournit un accès aléatoire et une mise à jour avec une complexité constante (effectively constant, pour être précis) ainsi que des opérations d’ajout et de concaténation très rapide* *[…]*
>   

Vous l’avez donc compris, Vector _ qui a été ajouté à Scala à partir de la version 2.8 _ est, grâce à sa puissance en terme de rapidité de traitement, aujourd’hui l’implémentation par défaut des séquences indexées immuables.



Et voici aussi quelques opérations basiques que vous pouvez essayer depuis vos machines :  

```scala
Scala> val MonVec1 = Vector(40,50,60)
# Vector(40, 50, 60)
Scala> val MonVec2 = MonVec1 :+ 70
# Vector(40, 50, 60, 70)
Scala> val MonVec3 = MonVec2 ++ Seq(80,82)
# Vector(40, 50, 60, 70, 80, 81, 82)
Scala> val MonVec4 = 30 +: MonVec3
# Vector(30, 40, 50, 60, 70, 80, 81, 82)
Scala> MonVec4(2)
# res0 : 50
```

Dans les exemples précédents j’ai utilisé des entiers. Mais Vector peut être utilisé également sur tout type de variables.





# Constant ? Effectively constant ? Quelle est la différence ?



 Nous avons remarqué dans le tableau que j’ai mis dans l’introduction ainsi que dans la définition d’un Vector la présence d’une expression qui pourrait être inconnue pour certains. Il s’agit de la complexité e.C Effectively Constant dont la traduction littérale en français donne : Effectivement constant.

Comme son nom l’indique, il s’agit d’une complexité assez proche de la classique O(n) (constante) mais les deux termes ne représentent pas la même chose. Néanmoins, nous allons considérer qu’il s’agit de la même complexité pour le moment et j’y retourner dans le paragraphe suivant avec plus de détails.





# Comment fonctionne réellement un Vector dans Scala?



Vous allez voir que la réponse de cette question est la solution de notre problématique principale.

La première chose à savoir c’est qu’un Vector est implémenté en Scala comme un arbre équilibré dont les nœuds possèdent chacune jusqu’à 32 fils et sont composés de 32 éléments.

On peut donc dire qu’un Vector est en vrai un arbre 32-aire (comme dans binaire)

Je vous laisse imaginer le modèle…

On constate alors que pour un Vector d’au maximum 32 éléments, un nœud suffit pour le représenter. Si on a entre 33 et 1024 (32*32) éléments, un passage au niveau suivant de l’arbre est obligatoire. Entre 1025 et 32768 (32*32*32) encore un passage, et ainsi de suite.




Maintenant, disons qu’on essaye de sélectionner un élément d’indice i dans un Vector de taille n (qui contient n éléments). L’algorithme va donc commencer à parcourir l’arbre nœud par nœud à la recherche de cet élément. Nous savons que dans un arbre binaire, au pire des cas (l’élément se retrouve à la dernière position) nous aurions à parcourir log2(n) nœuds. Un traitement pareil nous aurait coûté une complexité de O(log2(n)). Or, dans notre cas il s’agit d’un arbre 32-aire, nous devons dans le pire des cas parcourir log32(n) nœuds ce qui fait une complexité de O(log32(n)).

Comme nous le savons, beaucoup d’autres opérations sur les arbres binaires comme l’insertion, la suppression ou encore le remplacement d’un élément dans l’arbre engendrent une complexité de O(log2(n)) ([un petit rappel](http://mikefroh.blogspot.com/2011/03/immutable-binary-trees.html) pour l'insertion par exemple pour ceux qui ont oublié, ou encore [Wiki](https://fr.wikipedia.org/wiki/Arbre_B#Insertion) pour le tout). Nous pouvons donc déduire par la même démarche précédente que ces mêmes opérations coûtent O(log32(n)) dans un Vector !




À ce point, nous avons réussi à démontrer que presque toutes les opérations de base de Vector ont une complexité de O(log32(n)). Il est donc temps de clarifier le sens du terme Effectively Constant qu’on a utilisé plus haut dans le tuto. L’origine de cette appellation est assez simple :

Nous avons compris la structure d’un Vector et comment se fait le parcours du fameux arbre 32-aire. Supposons maintenant qu’on possède un Vector de 2 000 000 éléments.

La récupération de l’élément d’indice 1 000 000 coûte log32(1 000 000) = 3.986.

Maintenant, essayons de récupérer l’élément d’indice 2 000 000 qui est quand même 1 000 000 éléments plus loin du premier. La complexité n’est que de log32(2 000 000)=4.186.

La différence entre ces deux complexités est si petite, qu’elle peut être ignorée.

D’une façon générale, pour tout Vector d’une taille raisonnable, une sélection doit donner une complexité plus ou moins constante (pour un Vector de taille 215, la complexité est égale à 6 seulement), d’où l’aspect Effectively Constant de la complexité en Vector !





# Et pour finir, un peu de maths!



Ok ! Nous avons bien analysé les choses et nous avons compris que les opérations de base en Vector sont Effectively Constant et ont une complexité de O(log32(n)).  

Mais si je vous disais maintenant qu’en vrai cette complexité est équivalente à O(log2(n)) ?  

Pas de magie, simplement quelques connaissances en mathématiques (niveau terminal) et des notions en complexité :  

- Nous savons que pour tout n, log32(n) = logX(n) / logX(32) ,

- Prenons X=2, log32(n) = log2(n) / log2(32) ,

- Or, log2(32) = 5. D’ou, log32(n)=log2(n) / 5 = log2(n) * 0,2 ,

- Passons à la complexité, O(log32(n)) = O(log2(n) * 0,2) ,

- Or nous savons que pour toute constante K non nulle, O(K * f) = O(f) ,

- Donc, O(log32(n)) = O(log2(n)) .

  
  

Ainsi, nous avons démontré que les opérations de base en Scala sur les Vectors coutent O(log2(n)).

> Easy !





# Conclusion



À la fin, nous pouvons dire que la notion Effectively Constant a élargi les mesures standards de la complexité. Cette notion est basée sur des hypothèses de structure de données considérées comme non significatives pour avoir un impact sur la complexité. Il est juste, donc, d’affirmer que c’est un peu grâce à ce petit plus, que Vector séduit de plus en plus les développeurs Scala.




Nous arrivons à la fin de ce tuto. J’espère avoir été clair lors de mes explications. Merci d’avoir suivi jusqu’à la fin !``



​													





*													[MRABET Abderrahmen](https://github.com/MbtAbd/Scala), M2 EI2D, Promotion 2019 ®

------

