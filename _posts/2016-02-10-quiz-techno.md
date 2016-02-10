---
layout: post
title:  "Résultat du Quiz Techno 2015"
date:   2016-02-10 11:00:15
tags:
  - nerd-moment

author: fab
thumbnail: /images/quiz_techno.png
---

Pour célébrer l'arrivée de 2016, nous nous sommes amusés, Fabrice surtout, à faire un [quiz techno sur l'année 2015](https://servebox.typeform.com/to/F3zBVg)

Voici donc les bonnes réponses :

  * [la version courte avec les bonnes réponses sans explication](#short)
  * [la version longue avec explication et documentation à l'appui.](#long)

Et félicitation à Victor Laborie qui a eu 11 bonnes réponses.

Bonne lecture !

<span id="short">La version courte</span> avec seulement les bonnes réponses, pour ceux qui n'ont pas le temps (ou l'envie) de lire les explications

  1. Parmi ces 4 propositions, quelle phrase est correcte ?
  <br/>Je travaille à la NSA pour décrypter des messages

  2. Quel pays a organisé un hackathon pour développer des drones adaptés au cinéma ?
  <br/>La Nouvelle-Zélande

  3. Parmi ces 6 noms tirés de notre écosystème, lequel ne fait pas référence à la culture littéraire ?
  <br/>Jobz

  4. Quel auteur a cédé ses droits à Wikipedia en 2015 ?
  <br/>Antoine Bello

  5. Quel système informatique a piraté Lisbeth Salender en 2015 ?
  <br/>La NSA

  6. Une des pratiques suivantes n'a pas été utilisée chez ServeBox en 2015, laquelle ?
  <br/>Folk Programming

  7. Un éditeur de texte est devenu open source en 2015, lequel ? Indice : pas de danger qu'on l'utilise ;)
  <br/>Visual Studio

  8. Qu'avons-nous arrêté d'utiliser en 2015 ?
  <br/>Fig

  9. Que veut dire ES6, devenu un standard en 2015 ?
  <br/>ECMAScript 6

  10. Qui soutient le langage Rust, sorti en version 1.0.0 en 2015 ?
  <br/>Mozilla

  11. Qui soutient le langage Go, sorti en version 1.5 en 2015 ?
  <br/>Google

  12. Qui soutient le langage Elm sorti en version 0.15 le 20 avril 2015 ?
  <br/>Un gus dans un garage

  13. Quelle version de nodeJS n'est pas sortie en 2015 ?
  <br/>3.1.1

  14. Dans la version 2.3.0 de Ruby, par quoi a été remplacée la syntaxe suivante `u && u.profile && u.profile.thumbnails && u.profile.thumbnails.large` ?
  <br/>`u&.profile&.thumbnails&.large`

  15. Combien peut-on mettre de Raspberry Pi Zero dans les 21 racks composant le Eliott 405 (sorti 58 ans plus tôt) (l'image suivante peut vous aider) ?
  <br/>2,4 millions


<span id="long">Les questions ont été tirées des problématiques et informations qui ont égaillé notre année 2015 au sein de l'équipe technique.</span>


**1**

Le verbe crypter n'existe pas en français, ce qui enlève les phrases B et C.
<br/>Quand on envoie un message chiffré à une personne, cette personne le déchiffre avec sa clé.
<br/>La verbe décrypter est utilisé si on cherche à lire un message sans avoir la clé qui a chiffré le message.
<br/>C'est donc la phrase D qui est la bonne réponse.

   Le sujet de la vie privée et de la sécurisation des données est un sujet qui revient très régulièrement dans nos discussions techniques au sein de l'équipe. Jef a animé un meetup sur ce sujet et l'année dernière nous avons remis en place les clés GPG pour l'envoi des mails. Vous pouvez trouver les clés GPG des membres de ServeBox sur [MIT PGP Public Key Server](http://pgp.mit.edu/)


**2**

Dans l'équipe, Benjamin Sévérac fait de [l'urbex - (urban exploration)](https://www.flickr.com/photos/c0rben/) et est fan de drone (il en achète tous les 3 mois!). Il est donc normal que nous suivions l'actualité des drones et de la photo, pour cette raison le concours néo-zélandais [C-prize](http://www.cprize.nz/) fût un sujet pour nous.


**3**

L'erreur HTTP 451 fait référence à *Fahrenheit* 451 de Ray Bradury : [article Wikipedia : erreur HTTP 451](https://fr.wikipedia.org/wiki/Erreur_HTTP_451)
<br/>[Gros-Calin](https://github.com/servebox/gros_calin) *Gros-Câlin* est le premier *livre* de Romain Gary sous le pseudonyme d'Emile Ajar. Cette gem ruby nous sert pour afficher les données sur notre outil de supervision
<br/>[Taopaipai](https://github.com/servebox/taopaipai) Taopaipai est l'un des humains du manga Dragon Ball. Cette gem ruby nous sert à piloter un feu tricolore, qui affiche l'état de notre CI (Intégration Continue) : Vert : tout est bon, Rouge : des tests ne passent pas
<br/>[ElectricSheep](http://electricsheep.io) fait référence au livre *Do Androids Dream of Electric Sheep?* de Philip K. Dick, très connu pour son adaptation au cinéma *Blade runner*. ElectricSheep.IO est un projet open-source porté par ServeBox pour simplifier les sauvegardes de données
<br/>[Show&dont tell](https://servebox.com/show/) technique d'écriture employée entre autres par Ernest Hemingway [article Wikipedia : Show, don't tell](https://en.wikipedia.org/wiki/Show,_don't_tell)
<br/>[jobz](https://github.com/servebox/jobz) lui est un gem que nous avons développé pour nous faciliter l'envoi de commandes asynchrones en rails

**4**

Antoine Bello a donné ses droits d'auteur pour remercier Wikipedia, encyclopédie qui lui est très utile pour l'écriture de ses livres ([Pourquoi l'ecrivain Antoine Bello céde ses droit d'auteur 2014 à Wikipédia](http://www.usine-digitale.fr/article/pourquoi-l-ecrivain-antoine-bello-cede-ses-droits-d-auteur-2014-a-wikipedia.N353558))
<br/>Personnellement j'ai un petit faible pour *L'éloge de la pièce manquante*, Camille préférant la série des Falsificateurs

**5**

La sécurité des systèmes que nous mettons à disposition de nos clients est très importante pour nous.
Nous suivons au sein de l'équipe les infos concernant les failles, les piratages.
A part la NSA, toutes les autres réponses ont été piratées l'année dernière. Ce qui a pu entraîner des conséquences plus ou moins graves.

**6**

Le MOB programming est une technique de programmation en équipe. Je vous laisse lire [mobprogramming.org](http://mobprogramming.org/)
<br/>Le Pair programming, comme dans les avions. On met deux personnes pour coder : un pilote et un copilote
<br/>Code review : revue de code faite par une autre personne de l'équipe.
<br/>Remote programming : les développeurs ne sont pas toujours dans le même lieu physique. Il nous arrive donc de coder à distance en utilisant notamment tmux et emacs (voir vi)

**7**

Depuis quelques années Microsoft a décidé de rendre disponible du code source de certains de ses produits.
En 2010, il ont mis à disposition [F#](http://fsharp.org/), l'année dernière [Visual Studio](https://github.com/microsoft/vscode) et en début d'année [ChakraCore](https://github.com/Microsoft/ChakraCore) le moteur JS du navigateur Edge.

**8**

[Fig](http://www.fig.sh/) est la techno que nous avons arrêté d'utiliser et pour cause. Le projet a été racheté par docker, il est désormais connu sous le nom docker-compose.
Pour les 4 autres réponses ([Let's encrypt](https://letsencrypt.org/), [HTTP/2](https://http2.github.io/), [docker](https://www.docker.com/), [Elixir](http://elixir-lang.org/)), nous utilisons, expérimentons ou regardons du coin de l’œil ces technos.

**9 -> 12** Rien à ajouter

Et comme pour la réponse ci-dessus nous utilisons, expérimentons ou regardons du coin de l’œil ces langages ([ECMAScript 6](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf), [Rust](https://www.rust-lang.org/), [Go](https://golang.org/), [elm](http://elm-lang.org/)).
Il est possible, que pour de futures problématiques de nos clients, nous utilisions un de ces langages.

**13**

Nous utilisons nodejs dans notre chaîne de production, et la gestion de version de nodejs nous a beaucoup fait rire en 2015. Cette question était pour leur rendre hommage.
<br/>D'ailleurs depuis j'ai appris qu'il existe d'autres projets qui ont une gestion bien étrange des numéros de version. Par exemple Slackware est passé de la version 4 à 7 en 1999 pour avoir un numéro de version similaire à ses concurrents [article Wikipedia : Slackware](cf. https://fr.wikipedia.org/wiki/Slackware). Merci à [Jean-Michel ARMAND](https://twitter.com/mrjmad) pour cette anecdote.

**14**

Une question sur ruby devait figurer dans notre quizz techno. Une grande partie de notre temps de codage est sur des applications Rails.

**15**

Si vous voulez vérifier mes calculs je vous donne le lien que j'ai utilisé : [Raspberry Pi Zero vs Elliott 405](http://www.spinellis.gr/blog/20151129/)

----


**Et la question à laquelle vous avez échappée, qui a animé les débats au sein de l'équipe.**

  Comment traduit-on happening en français ?

  1. Événement
  2. Evénement
  3. Evènement
  4. Èvènement
  5. Événement

Je vous laisse trouver la (les) bonne(s) réponse(s) avec les 2 liens suivants :

   * [article Wikipedia : Événement](https://fr.wikipedia.org/wiki/%C3%89v%C3%A9nement) ;
   * [article Wikipedia : Usage des majuscules en français](https://fr.wikipedia.org/wiki/Usage_des_majuscules_en_fran%C3%A7ais#Accentuation_des_majuscules_et_des_capitales)
