# La gestion du temps avec Eve
## L'enregistrement #time
La gestion du temps dans Eve se fait à travers l'enregistrement `[#time]`, qui nous sert d'interface pour accéder à l'horloge système.

```eve
search
  [#time year month day hours hours-24 minutes seconds time-string frames ampm]

bind @browser
  [#div, text: "year: {{year}}"]
  [#div, text: "month: {{month}}"]
  [#div, text: "day: {{day}}"]
  [#div, text: "hours: {{hours}}"]
  [#div, text: "hours-24: {{hours-24}}"]
  [#div, text: "minutes: {{minutes}}"]
  [#div, text: "seconds: {{seconds}}"]
  [#div, text: "time-string: {{time-string}}"]
  [#div, text: "frames: {{frames}}"]
  [#div, text: "ampm: {{ampm}}"]
```

Théoriquement, `[#time frames]` est mis à jour toutes les 16 millisecondes, mais ce n'est pas le cas avec la version master de Eve au moment où je vous parle (ou frame est mis à jour une seule fois par seconde).

## Programmer un événement
Comme pour tous les enregistrements, il est possible de faire des recherches sur l'enregistrement `#time`, et c'est ce que nous allons utiliser pour programmer un événement.
Notre objectif ici sera d'afficher à l'écran "Hello world" pendant une seconde, et ce à chaque nouvelle minute.

```
search
  time = [#time seconds: 0]
  
commit @browser
	[#div, text: "Hello world", time: time]
```

Notez la référence à l'enregistrement time qui a déclenché notre "événement". Sans lui, il serait impossible de créer notre enregistrement une seconde fois, puisqu'il est impossible de créer deux enregistrements parfaitement identique.

# Créer des timers
Si programmer un événement est ici quelque chose de bien plus simple que dans la plupart des langages, créer un timer est quelque chose d'un petit peu plus complexe.
Pour commencer, créons nos enregistrements. Ils ont besoin d'un tag `#timer` et d'un attribut remaining-seconds pour savoir dans combien de temps ils vont se déclencher.

```
commit
  [#timer #doSomething, remaining-seconds: 10]
  [#timer #doSomethingElse, remaining-seconds: 25]
```

Pour être capable de suivre ce qui se passe en interne, affichons donc nos timers.

```
search
	timer = [#timer]
  
bind @browser
	[#div, text: "timer : {{timer.remaining-seconds}} --- update: {{timer.last-update}}"]

```

## Mécanique
Notre objectif est de pouvoir décrémenter les remaining seconds de 1 toutes les secondes tant que nous n'avons pas encore atteint 0.
Or, si nous décidons d'y aller naïvement, nous pouvons noter que le programme s'emballe et selon les versions ou se mettra soustraire environ 60 toutes les secondes, ou plante tout simplement.

```eve disabled
search
	time = [#time seconds]

	timer = [#timer remaining-seconds]
  not(timer = [remaining-seconds: 0])

commit
	timer <- [remaining-seconds: remaining-seconds - 1]
```

Le problème vient du nombre de fois que #time est appelé, qui n'est malheureusement pas d'une fois par seconde.
Pour pallier le problème, ajoutons simplement à l'enregistrement la seconde ou a ete fait la dernière mise à jour.

```eve
search
	time = [#time seconds]

	timer = [#timer remaining-seconds]
  not(timer = [remaining-seconds: 0])
  not(timer = [last-update: seconds])

commit
	timer <- [remaining-seconds: remaining-seconds - 1, last-update: seconds]
```

La dernière étape est évidemment de mettre en place l'action à effectuer à la fin du timer. Pour cela, nous capturons les timers en fin de vie en fonction de leurs tags supplémentaires, et y ajoutons une action à effectuer.

```
search
	timer = [#timer #doSomething, remaining-seconds: 0]
  
commit @browser
	[#div, text: "#doSomething est fini !"]
  
commit
  timer := none
```

Notez le `timer : = none` dans notre bloc, qui permet de détruire notre enregistrement, maintenant qu'il est inutile. Il aurait été possible de redonner une valeur à remaining-seconds, pour faire en sorte que l'événement se répète.

# Aller plus loin
Ce système de timers est pleinement fonctionnel mais très perfectible. Si le sujet vous intéresse, que diriez-vous d'un exercice pratique ?
Il manque selon moi deux fonctionnalités importantes à notre timer :

* Pouvoir déterminer le temps avant déclenchement en seconde, minutes et heures avec un enregistrement du style [`#timer, remaining-seconds: 30, remaining-minutes: 10]`.
* Pouvoir relancer un événement simplement, sans avoir à rerentrer le temps à chaque fois s'il est identique.

Manifestez-vous en commentaires si vous avez trouve une solution à ce petit exercice !
