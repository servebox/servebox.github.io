---
layout: post
title:  "In bed with TLS - Partie I : TLS, PFS et Logjam"
date:   2015-06-29 08:00:15
tags:
  - tech
author: jef
thumbnail: /images/logjam.png
---

Fin mai, des chercheurs, dont des chercheurs de _l'INRIA_ (cocorico), ont dévoilé, sous le nom de _Logjam_, des défauts de sécurité impactant TLS. J'avais pensé écrire une série de billets là dessus, ce que je n'avais malheureusement pas eu loisir de faire jusqu'ici.

Ce billet vise à illustrer le fonctionnement de TLS et l'échange de clés éphémères _Diffie-Hellman_ (DHE). J'y parle ensuite de la "récente" faille _Logjam_. Même si je les aborde, j'ai essayé de réduire au minimum les aspects mathématiques des méthodes cryptographiques entrant en jeu et de ne pas rentrer dans le détail des protocoles ou de la configuration. L'Interweb regorge en effet de howtos et autres articles à ces effets. Écrire est aussi un moyen pour moi de mettre mes idées au clair. Bref, c'est parti !

## Chiffrement et cryptographie

Quand elle pense _cryptographie_, la plupart d'entre nous pense au _chiffrement_, c'est à dire à la transformation d'un message - une donnée en clair - en un texte illisible pour qui ne possède pas la clé de _déchiffrement_ (à partir de ce point, quand je parle de message, je parle de l'échange de données entre deux machines). C'est en effet l'une des finalités recherchées, mais en réalité, la cryptographie remplit différentes fonctions, notamment :

* **la signature**, qui permet à l'émetteur d'un message de le signer cryptographiquement et, par réciproque, au destinataire de vérifier l'authenticité du message ;
* **le contrôle d'intégrité** qui permet de vérifier qu'un message n'a pas été modifié ou corrompu pendant son acheminement ;
* **la confidentialité**, qui permet de rendre secrets les messages échangés par _chiffrement_.

L'ensemble de ces fonctions est généralement à considérer quand il s'agit d'établir des modes de communications sécurisés entre deux machines. La multiplicité des fonctions à appréhender et leur empilement font d'ailleurs partie, à mon humble avis, des principales difficultés pour appréhender la problématique.

## Symétrique et asymétrique

Lorsque l'on parle de cryptographie, deux types d'approche sont aujourd'hui employés.

La première est la plus intuitive, il s'agit de la **cryptographie symétrique** qui permet à deux machines d'échanger des messages, dès lors qu'elles partagent toutes les deux un "code" secret : **une clé partagée**. Les algorithmes de chiffrement symétriques, notamment AES ([Advanced Encryption Standard](https://fr.wikipedia.org/wiki/Advanced_Encryption_Standard)), sont efficaces et performants pour le chiffrement de gros messages, mais présentent l'inconvénient majeur de nécessiter, comme préalable au chiffrement, **le partage d'une clé secrète identique** entre les interlocuteurs.

La seconde, **la cryptographie asymétrique**, repose sur la relation mathématique établie entre **deux clés distinctes**. Une clé publique, d'une part, qui peut être librement diffusée, et une clé privée, d'autre part, connue seulement par l'un des deux interlocuteurs. Ce type de cryptographie est particulièrement pratique dans la mesure où l'une des clés peut-être rendue publique, à des fins de chiffrement ou d'authentification, sans risque de compromission du système. Parmi les algorithmes utilisés, c'est [RSA](http://fr.wikipedia.org/wiki/Chiffrement_RSA) qui domine aujourd'hui (même si un algorithme basé sur les "courbes elliptiques" s'implante progressivement : _ECDSA_).

## Le protocole TLS

TLS (pour [Transport Layer Security](http://fr.wikipedia.org/wiki/Transport_Layer_Security)), est l'un des protocoles de cryptographie point à point le plus utilisé sur Internet, notamment pour sécuriser les communications entre un navigateur et un serveur Web, mais aussi pour l'échange d'emails (SMTP avec StartTLS, IMAPS, POP3S) ou l'établissement de connexions sécurisées (SSH, VPN). Pour garantir la sécurité des échanges entre un client et un serveur, TLS essaye de répondre à deux problématiques cruciales : l'authentification et la confidentialité. Son rôle est de faire en sorte que, grâce à _une poignée de main_ (ou _handshake_), les deux parties (le client et le serveur) se mettent d'accord sur les modalités de sécurisation de leur connexion. Dans l'exemple qui suit nous assumerons qu'il s'agit de TLS pour le Web, mais cela serait facilement transposable pour d'autres protocoles.

La première nécessité est de s'assurer qu'une autre machine, aux mains d'un indélicat, n'essaiera de se faire passer pour le serveur. TLS prévoit que le serveur transfère au client, lors des premières étapes, **un certificat contenant sa clé publique**. Le certificat est signé cryptographiquement par une autorité de certification (CA), le client peut ainsi contrôler sa validité vis-à-vis du nom de domaine du serveur.

L'objectif de TLS est de rendre confidentiels les messages qui seront échangés au cours de la session. Comme il s'agit de messages de taille importante, c'est du **chiffrement symétrique** qui sera utilisé _in fine_. Or, nous l'avons vu, ce type de chiffrement impose qu'une clé secrète soit partagée entre le client et le serveur. Le client génère donc une clé aléatoire, la _pre-master key_ (ou PMK), qu'il chiffre à l'aide de la clé publique contenue dans le certificat fourni par le serveur, et la transmet à ce dernier. Le serveur utilise sa clé privée pour déchiffrer le message et en extraire la clé. Le client et le serveur étant tous deux en possession de la PMK, chacun d'entre eux peut se baser dessus, la _dériver_, pour créer une clé de chiffrement symétrique qui sera utilisée pour la session. Par ailleurs, si le serveur est capable de déchiffrer la PMK, c'est qu'il est en possession de la clé privée correspondant à la clé publique du certificat. En effet, en chiffrant la PMK, TLS fait d'une pierre deux coups : il permet le partage de la clé de chiffrement symétrique qui assurera **la confidentialité**, mais également, et c'est tout aussi important, **l'authentification** du serveur.

Si nous laissons de côté la signature du certificat du serveur par l'autorité de certification, les différentes fonctions du protocole seront mises en oeuvre via la combinaison suivante :

<table>
<tr><th>Fonction</th><th>Système</th></tr>
<tr><td>Authentification</td><td>RSA</td></tr>
<tr><td>Agrément Pre-Master Key</td><td>RSA</td></tr>
<tr><td>Chiffrement de la session</td><td>Chiffrement symétrique (AES)</td></tr>
</table>

On peut noter que les fonctions cruciales d'authentification **et** de chiffrement dépendent toutes deux de la sécurité de la clé privée du serveur. Si cette clé est compromise, l'attaquant sera en mesure de déchiffrer le trafic futur, ou de se faire passer pour le serveur sans que les clients ne se doutent de rien. Mais l'autre problème de TLS utilisé de cette façon, c'est que l'attaquant pourra aussi **déchiffrer les échanges passés**. En effet, toutes les sessions passées étaient protégées par une clé dérivant de la _pre-master key_, elle-même chiffrée à l'aide de la clé publique _RSA_. Pour peu que l'attaquant ait enregistré des échanges, il sera capable de les déchiffrer. Pour les personnes gérant le serveur, impossible de corriger le tir : même en changeant la clé privée sur le serveur, rien n'y fera, les échanges passés seront considéré comme compromis.

## Forward Secrecy

La _forward secrecy_ (souvent appelée _perfect forward secrecy_, ou PFS) permet de rendre la confidentialité d'un échange persistante. Autrement dit, de s'assurer qu'un échange chiffré à un instant T, restera confidentiel dans le futur, même en cas de compromission de la clé privée du serveur.

Pour ce faire, il faut décoreller l'échange de la clé de chiffrement symétrique de la paire de clés asymétrique servant à l'authentification du serveur.

L'échange de clés éphémères _Diffie-Hellman_ (DHE) vise à satisfaire cette condition. Il se base sur le fait qu'en mathématiques les exposants sont _commutatifs_. Plus simplement, si vous élevez un nombre _g_ à la puissance _a_ (**A = g<sup>a</sup>**), et le résultat à la puissance _b_ (**A<sup>b</sup> = g<sup>ab</sup>**), vous obtiendrez le même résultat qu'en élevant d'abord _g_ à la puissance _b_ (**B = g<sup>b</sup>**), puis le résultat à la puissance _a_ (**B<sup>a</sup> = g<sup>ba</sup> = g<sup>ab</sup> = A<sup>b</sup>**).

L'échange de clés fonctionne en faisant effectuer à chacune des machines une partie du calcul :

* le serveur tire au sort un nombre _a_ qu'il ne communique pas, calcule sa "partie" de la clé (_A = g<sup>a</sup>_) et la transmet au client ;
* le client, à son tour, tire au sort un nombre _b_ qu'il garde pour lui, calcule l'autre partie de la clé (_B = g<sup>b</sup>_) et la transmet au serveur ;
* le serveur, connaissant _a_ et la partie de la clé du client (_B_), peut recomposer la clé, la _pre-master key_ (_B<sup>a</sup> = g<sup>ba</sup>_) ;
* le client, connaissant _b_ et la partie de la clé du serveur (_A_), peut à son tour recomposer la _pre-master key_ (l'équivalent de _A<sup>b</sup> = g<sup>ab</sup>_) ;
* à partir de la PMK (qui est identique de part et d'autre), le client et le serveur peuvent dériver la clé de chiffrement symétrique pour la session.

Les paramètres privés aléatoires, _a_ et _b_, changent à l'établissement de chaque session et ne sont jamais stockés. Si un attaquant écoute l'échange, il ne verra jamais la PMK complète (_g<sup>ab</sup>_), seulement les valeurs de l'échange Diffie-Hellman (_g<sup>a</sup>_ et _g<sup>b</sup>_). Pour calculer _g<sup>ab</sup>_, il lui faudra déduire _a_ de _g<sup>a</sup>_ ou _b_ de _g<sup>b</sup>_. Ce n'est pas très compliqué de le faire avec des nombres courants, alors pour donner du fil à retrordre aux cryptanalystes, DHE utilise un grand nombre premier _p_ avec lequel sont divisés _g<sup>a</sup>_ et _g<sup>b</sup>_. Ce sont les restes de chaque division (_g<sup>a</sup> modulo p_ et _g<sup>b</sup> modulo p_) qui sont en réalité échangés entre le client et le serveur. Pour déduire les secrets, l'attaquant devra résoudre le problème du _logarithme discret_.  On ne connaît à ce jour pas de méthode efficace à ce problème, pour peu que le groupe Diffie-Hellman utilisé, c'est à dire le nombre de valeurs possibles de _p_, soit suffisamment grand.

Pour que le client puisse l'authentifier, le serveur signe cryptographiquement les messages de la _poignée de main_ grâce à sa clé privée _RSA_. Cette fois-ci, les fonctions sont donc combinées de cette façon :

<table>
<tr><th>Fonction</th><th>Système</th></tr>
<tr><td>Authentification</td><td>RSA</td></tr>
<tr><td>Agrément Pre-Master Key</td><td>DHE</td></tr>
<tr><td>Chiffrement de la session</td><td>Chiffrement symétrique (AES)</td></tr>
</table>

Contrairement à TLS lorsqu'il est utilisé sans DHE, l'échange de clés Diffie-Hellman permet de répartir les responsabilités : la sécurité de l'agrément sur la _pre-master key_ ne repose plus sur la clé privée du serveur. Si cette dernière est compromise, l'attaquant pourra se placer entre le client et le serveur pour se faire passer pour ce dernier (_Man in The Middle_) ; en revanche, **il ne pourra pas déchiffrer les sessions passées**, les eût-il enregistrées.

Il est également possible, pour l'échange de clés Diffie-Hellman, de se baser sur les courbes elliptiques. On parle d'Échange Diffie-Hellman sur une Courbe Elliptique (ECDHE). Le principe de l'échange reste identique - le client et le serveur effectuent chacun une partie du calcul permettant de s'accorder sur une _pre-master key_, mais l'on se base dans ce cas sur une courbe elliptique et plus sur un groupe de nombres premiers.

## Logjam

Lors de la _poignée de main TLS_, le client indique au serveur les combinaisons d'algorithmes qu'il supporte pour l'agrément sur la _pre-master key_, l'authentification, le chiffrement symétrique, et la fonction de hachage utilisée pour le contrôle d'intégrité des messages. Par exemple, pour un échange de clé Diffie-Hellmann (`DHE`), une authentification _RSA_ (`RSA`), un chiffrement  symétrique AES en 128 bits (`AES128-GCM`) et SHA en 256 bits comme fonction de hachage pour le calcul des _MAC_ (`SHA-256`), la _cipher suite_ sera nommée `DHE-RSA-AES128-GCM-SHA256`. Le client propose donc la liste des suites d'algorithmes qu'il supporte, et le serveur choisit à partir de cette liste la combinaison qu'il préfère.

Or, les États-Unis souhaitaient auparavant limiter l'export des technologies de cryptographie. Pour pouvoir exporter leurs logiciels, les concepteurs avaient été contraints de créer des versions faibles des logiciels de cryptographie. La version faible de l'échange de clé Diffie-Hellman (`DHE_EXPORT`) limite la taille du groupe Diffie-Hellmann (la taille potentielle de notre nombre premier _p_) à 512 bits. La législation U.S. a depuis changé, mais certains logiciels continuent de supporter `DHE_EXPORT`, ce qui permet à un attaquant se plaçant entre le client et le serveur, de négocier avec le serveur, au nom du client, un agrément basé sur cette version dégradée de DHE.

Le second problème révélé par _Logjam_, est lié à la manière dont est choisi le groupe Diffie-Hellman. Pour casser un échange Diffie-Hellman, il faut résoudre le problème du logarithme discret, nous l'avons évoqué plus haut. La méthode connue la plus efficace est l'algorithme du _crible de corps de nombres_ (ou NFS pour _Number Field Sieve_). Cet algorithme fonctionne grosso modo en deux étapes. La première et la plus lourde, celle de précalcul, dépend de _p_, notre groupe Diffie-Hellman. La seconde, la _descente_, permet de calculer le logarithme discret. L'effort nécessaire pour cryptanalyser DHE concerne essentiellement l'étape de précalcul, qui doit être répétée pour chaque valeur différente de _p_. Ainsi, **la sécurité de DHE repose sur la taille des groupes Diffie-Hellman**, d'une part (plus les valeurs de _p_ potentielles sont nombreuses, plus le _NFS_ sera long et coûteux), **mais aussi sur leur originalité**. Si chaque serveur possède un groupe Diffie-Hellman qui lui est propre, l'attaquant devra refaire le travail à chaque fois, ce qui est en pratique impossible.

Si, en revanche, un grand nombre de serveurs partagent le même groupe Diffie-Hellman, une organisation bien équipée en puissance de calcul, comme un gouvernement, pourrait tout à fait stocker le résultat de la phase de précalcul dans une base de données. Cette dernière peut être arrangée, en effet, pour ne dépendre que de _p_, le groupe _DH_. Pour casser des échanges Diffie-Hellman, il ne restera plus que l'étape de _descente_ qui, elle, est beaucoup plus rapide. C'est exactement ce qu'ont révélé les chercheurs à l'origine de _Logjam_. Pour une vision plus détaillée, je vous conseille la lecture de [cet excellent article](http://www.scottaaronson.com/blog/?p=2293) (en anglais), ou du [papier publié pour l'occasion](https://weakdh.org/imperfect-forward-secrecy.pdf).

A ce jour, **l'échange de clé Diffie-Hellman est donc fiable lorsque il est correctement configuré**. La double possibilité d'attaque révélée par [Logjam](https://weakdh.org/) exploite la manière dont DHE est parfois déployé en pratique.

Pour se prémunir de Logjam, **c'est très simple**, quelques manips et lignes de configuration suffisent, les personnes gérant des serveurs peuvent:

* mettre à jour les logiciels serveurs vers les versions les plus récentes (OpenSSH en particulier);
* générer **un groupe Diffie-Hellman original et de taille suffisante** (2048 ou 4096 bits) ;
* n'accepter que des _suites de cipher_ considérées comme fiables, exclure en particulier `DHE_EXPORT` ;
* utiliser des outils de test permettant d'évaluer si la configuration de leur serveur est correcte.

Tout celà est très bien expliqué, par exemple [ici](https://weakdh.org/sysadmin.html) ou [là](http://jlecour.github.io/ssl-gandi-nginx-debian/).

Si vous êtes arrivés jusqu'ici, il est probable que vous ayez trouvé deux ou trois informations pertinentes. Dans la suite de la série, je vous propose donc d'élargir en nous intéressant à un sujet connexe : les autorités de certification.

_Note : originalement publié sur [mon blog personnel](http://http://nonblocking.info/in-bed-with-tls-part1/)._
