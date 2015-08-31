---
layout: post
title:  "Cryptographie de comptoir"
date:   2015-07-19 10:44:03

author: jef
thumbnail:  /images/chiffrement-bout-en-bout-500.png


---
L'actualité des dernières années, des révélations d'Edward Snowden sur la NSA aux tristes attentats projetés ou perpétrés en France et ailleurs, en passant par les débats sur la Loi Renseignement, donnent progressivement une exposition plus grande à la crypto dans les médias. Pour le meilleur, et surtout pour le pire.

La pression politique s'accentue sur la cryptographie au nom de la "sécurité". Les partisans, en France et ailleurs, d'une cryptographie plus lourdement régulée, associent à tort la cryptographie à la seule confidentialité des échanges. Certains sont littéralement obnubilés par le fait que, potentiellement ou factuellement, des organisations criminelles ou terroristes puissent communiquer de manière confidentielle. Selon eux, la cryptographie devrait être donc être affaiblie et son usage placé sous un contrôle accru des services de l'État. C'est oublier que dans une société désormais largement dépendante, sur nombre de questions, de moyens de communications "numériques", la cryptographie est à peu près notre seul moyen pour éviter que tout le machin tourne au merdier généralisé.

Dans ce contexte, il n'est pas rare que des ~~trolls~~ geeks et autres hackerz reprennent ici ou là un journaliste, un juriste ou un politique sur son usage d'un mot ou d'une expression incorrects (j'y reviendrai), comme "crypter", "cryptage", "crypté" ou autres termes issus du grec _crypta_. Récemment, j'ai pris part à un certain nombre d'échanges de ce type, notamment sur Twitter. Dans l'un d'entre eux, un journaliste annonçait qu'il venait de participer à une formation dispensée par un de ses confrères : "Crypter ses échanges et protéger ses sources". Après coup, j'avoue m'être senti un peu con de critiquer la forme de ce qui est fondamentalement une initiative essentielle. Qu'un journaliste convainque d'autres journalistes d'utiliser des outils et de techniques permettant de protéger leurs sources, qu'il les y forme de surcroît, est indéniablement très positif. Néanmoins, je reste convaincu que les approximations sémantiques ou logiques sont nuisibles, dans la mesure où la cryptographie est omniprésente dans notre vie quotidienne, que son importance ne fait que croître pour garantir notre sécurité et nos libertés, et que l'on assiste à une offensive politique et médiatique internationale contre son libre usage.

## Petit lexique du voyageur cryptographique

Si l'on souhaite assurer la **confidentialité** d'un échange, un contenu en "texte clair", on assurera sa transformation en "texte chiffré" par l'opération dite de **chiffrement**, effectuée à l'aide d'une clé. À l'autre bout, le ou les destinataires du message récupèreront le contenu original, le rendront lisible, grâce à un **déchiffrement** effectué lui aussi à l'aide d'une clé (légitime donc, suivez un peu). Si une personne ne possédant pas la clé de déchiffrement souhaite accéder au contenu "en clair", il lui faudra opérer un **décryptage** ; le but de la cryptographie étant de rendre le chiffrement et le déchiffrement faciles, et le décryptage impossible.

![Vivons d'amour et de crypto fraîche](http://nonblocking.info/content/images/2015/07/chiffrement-bout-en-bout-1200.png)

Nous obtenons donc :

<table>
<tr>
<td>Chiffrement</td>
<td>Transformation d'un texte clair (lisible) en un texte chiffré (illisible pour qui ne possède pas la clé de déchiffrement)</td>
</tr>
<tr>
<td>Déchiffrement</td>
<td>Transformation d'un texte chiffré en texte clair, <strong>à l'aide de la clé de déchiffrement</strong></td>
</tr>
<tr>
<td>Décryptage</td>
<td>Transformation d'un texte chiffré en texte clair, <strong>sans posséder la clé de déchiffrement</strong></td>
</tr>
</table>

"Crypter" ne fait pas partie de ce tableau, pour une raison toute simple : il n'a absolument aucun sens. Il impliquerait en effet une forme de chiffrement sans clé, ce qui rendrait le texte "crypté" **définitivement** inutilisable, et ce pour qui que ce soit. Quand je vois la simplicité de ce lexique, j'ai un peu de mal à comprendre en quoi l'utilisation du terme "chiffrer" est si difficile à admettre ou à populariser.

## Cryptage, cryptement, cryptation, crispation

Alors vous me direz que, si "chiffrer" est tout à fait envisageable, l'utilisation de "crypter" ne pose dans le fond pas de problème. Le terme serait même bien compris de la plupart des Gens™, toussa. Bien, bien, bien...

La vocation historique de la cryptographie est la dissimulation de secrets (si possible relatifs à des projets d'assassinat, des conflits politiques, des adultères, et autres histoires de fesses hardcore). Or, le concept de confidentialité est _de facto_ étendu par la diversité et l'utilisation extensive des moyens de communication modernes, d'une part. D'autre part, la cryptographie moderne prend couramment en charge  d'autres fonctions. **L'authentification** permet de vérifier _l'identité_ des personnes ou des machines à l'origine des messages. Le contrôle de **l'intégrité** des communications permet quant à lui d'assurer que celles-ci n'ont pas été altérées - modifiées - au cours de leur acheminement sur les réseaux. Craignez-vous davantage que quelqu'un intercepte vos communications pour les lire, ou qu'un usurpateur puisse se faire passer pour vous pour insulter votre meilleur(e) ami(e), votre patron ? Accepteriez-vous que quelqu'un puisse modifier le contenu de vos échanges, le montant ou la destination d'un virement bancaire ? Le verbe "crypter" est de ce point de vue une simplification à outrance, qui évoque la cryptographie sous le seul angle de la **confidentialité**. Cela nuit fondamentalement à la compréhension, même superficielle, du sujet.

![Cryptologie](http://nonblocking.info/content/images/2015/07/crypto-tree-1280.png)

Pire, qu'il s'agisse de sensationalisme délibéré, d'ignorance ou de paresse intellectuelle, ou encore de sincères et regrettables maladresses, utiliser "crypter" c'est évoquer des pratiques sombres, honteuses, sales et cachées. Or, la cryptographie sous ses différentes formes est d'abord une science dont l'objectif est tout au contraire d'apporter des garanties de liberté **et** de sécurité. Il y a dans ces errements sémantiques quelque chose du registre de la sorcellerie, de la magie, et de la peur. Et puisqu'on est plus à ça près, je vous propose de prendre une gorgée d'Umberto Eco :

> L'abbé bénédictin Trithemius a été, au XVè siècle, l'un des précurseurs de la cryptographie moderne : il élaborait des systèmes de codification secrète pour instruire les gouvernants et les chefs d'armées, mais, afin de susciter l'interêt pour ses découvertes (...), il montrait que sa technique était en fait une opération magique grâce à laquelle on pouvait convoquer les anges qui, en une seconde, portaient au loin, et en secret, nos messages.

Dans ces "crypter", "cryptage", "crypté", qui truffent les déclarations politiques, les communications publiques ou les articles de presse, c'est la magie qu'on invoque. Là où l'on devrait trouver de l'information, on trouve une déformation qui arrange bien une classe politique totalement à l'arrache, qui se défausse de ses lourds échecs en toutes occasions en accablant un coupable facile à diaboliser : le démoniaque Internet.

Bien sûr, le langage est toujours affaire de convention ; on se met d'accord sur les termes et sur leurs objets. Le souci, voyez-vous, c'est que, en ce qui concerne la cryptographie, cette convention existe déjà depuis un petit bail. On ne vous a pas vraiment attendus, pour vous le dire autrement. En France, qu'on se le dise, Mesdames, Messieurs, on chiffre et on déchiffre. Les méchants, eux, essayent de décrypter. Crypter, ça n'existe pas. Point. Et non, en ce domaine, le Larousse ou le Petit Robert ne font pas autorité. En employant de mauvais termes, on élargit le fossé, on amplifie le décalage, entre ceux qui savent, qui fabriquent, qui conseillent, et ceux qui _in fine_ utilisent.

Alors à ceux qui _cryptent_ à tour de bras, permettez-moi de vous dire un truc, concernant ces "ceux-qui-utilisent". Ils ont tout autant le droit de savoir que vous ou moi, et le même besoin de comprendre.

Bisous.

_Note : originalement publié sur [mon blog personnel](http://nonblocking.info/cryptographie-de-comptoir/)._
