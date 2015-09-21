---
layout: post
title:  "In bed with TLS - Partie II : PKIX, DANE et HPKP"
date:   2015-09-20 10:20:01
tags:
  - tech
author: jef
thumbnail: /images/confiance.jpg
---
Dans [l'épisode précédent](/2015/06/29/in-bed-with-tls-1.html), nous avions profité de la récente actualité _Logjam_, pour faire un petit tour de _TLS_ en général, et de l'échange de clés Diffie-Hellman en particulier. Nous avions très brièvement évoqué la problématique de la certification. Je vous propose de l'examiner ici plus en détail ainsi que, plus généralement, l'infrastructure à clés publiques.

## Certification et PKI X.509

Si l'on souhaite établir une connexion sécurisée avec un serveur, il convient de s'assurer qu'un hypothétique attaquant ne se sera pas placé entre le client et le serveur pour se faire passer pour ce dernier - ce que l'on appelle une [attaque de l'homme du milieu](https://fr.wikipedia.org/wiki/Attaque_de_l%27homme_du_milieu) ou _MitM_, pour _Man-in-the-Middle_.

Nous l'avons vu précédemment, pour s'authentifier, le serveur fournit sa clé publique dans un certificat. En déchiffrant un message chiffré par le client à l'aide de cette clé publique (dans le cas de _TLS_ sans échange de clés Diffie-Hellman), ou en signant certains messages de l'échange de clés Diffie-Hellmann, le serveur apporte une **preuve de connaissance**, l'assurance qu'il possède la clé privée correspondant à la clé publique. Néanmoins, il ne démontre pas son "identité", sa légitimité à utiliser telle ou telle paire de clés. En l'absence d'une autre vérification, n'importe quel attaquant pourrait donc générer une paire de clés valide d'un point de vue mathématique.

Pour éviter ce problème, la clé publique fournie par le serveur est signée cryptographiquement par une **autorité de certification** (ou _CA_, pour _Certificate Authority_), un **tiers de confiance** qui vérifie, de manière plus ou moins poussée <sup>[1](#ca-validations)</sup>, l'identité de l'opérateur (personne physique ou organisation) d'un serveur. Le certificat du serveur contient donc sa clé publique, des informations d'identification (le nom du serveur et de l'organisation en particulier), ainsi que la signature numérique appliquée par l'autorité de certification sur l'ensemble. Lorsque le serveur fournit le certificat, il "suffit" au client de vérifier qu'il ait été signé par une autorité de certification connue.

Il convient ainsi que les clients connaissent les clé publiques des autorités de certification. Pour cette raison, les navigateurs web et autres logiciels utilisant TLS embarquent les certificats - les clés publiques - d'autorités dites "racines" de _l'infrastructure à clés publiques_ (ou PKIX pour _Public Key Infrastructure X.509_ <sup>[2](#pki-x509)</sup>). Les clés privées de ces autorités racines sont utilisées pour signer cryptographiquement les certificats d'autorités de certification "subordonnées" qui, à leur tour, utilisent leur clés privées pour signer d'autres certificats. Chaque certificat appartient donc à une chaîne, une hiérarchie, dans laquelle la confiance se propage d'une autorité de certification racine vers les feuilles (les serveurs).

![Hiérarchie des certificats de www.google.com](/images/google-certificate-chain.png)

## Anarchy in the PKIX

En pratique, il est difficile d'imaginer que les utilisateurs ne soient contraints de vérifier, par eux-mêmes, la validité de tel ou tel certificat, l'identité de tel ou tel serveur. C'est la raison d'être des infrastructures à clés publiques, nous l'avons évoqué ci-dessus : rendre **transparent** pour l'utilisateur l'ensemble des étapes de vérification. Toute cette infrastructure repose donc sur la confiance.L'utilisateur fait confiance à son logiciel client - typiquement son navigateur. Ce dernier fait lui-même confiance aux autorités de certification racines de la PKIX (_Public Key Infrastructure X.509_) qui, à leur tour, font confiance aux autorités subordonnées, et ainsi de suite.

![](/images/confiance.jpg)
<a href="https://en.wikipedia.org/wiki/Robert_Surcouf" class="caption">Capture du Kent par le Confiance</a>

Or, la _PKIX_ souffre de faiblesses qui, sans verser dans le dramatique ou la paranoïa, incitent à limiter le degré de confiance que l'on peut y mettre. Tout d'abord, il existe plusieurs centaines d'autorités de certification reconnues, ce qui rend l'inventaire et le contrôle des pratiques difficile. Pire, n'importe laquelle de ces autorités de certification **peut émettre un certificat pour n'importe quel domaine**. Je fais plutôt confiance à l'entreprise auprès de laquelle j'acquiers mes certificats (Gandi) mais cette confiance n'a que peu de valeur, dans la mesure où n'importe quelle autorité peut émettre des certificats pour mes noms de domaines. Tout récemment, [Symantec s'est ainsi fait attraper](https://boingboing.net/2015/09/19/symantec-caught-issuing-rogue.html) en train d'émettre des certificats valides sur le plan cryptographique mais totalement illégitimes. Pour le domaine `google.com`, excusez du peu. L'excuse invoquée est, c'est assez amusant, très similaire à celle d'une autorité de certification liée à l'ANSSI française, ou à celle de l'autorité de certification turque Turktrust qui, en leur temps, s'étaient aussi fait prendre la main dans le pot de confiture.

> Ah, mais ouais mais en fait non, on a pas fait exprès, on faisait des tests, LOL ! Haha, vous inquiétez pas, hein ! X) ptdr

Si je ne respecte pas la lettre, c'est fidèle à l'esprit... Accidents ou pas accidents, erreurs ou pas, il n'en reste pas moins que ces cas illustrent la relative vulnérabilité de la _PKIX_. Le piratage des autorités de certification est également un risque tout ce qu'il y a de réel ; on pense ainsi aux attaques subies par Comodo ou DigiNotar qui ont été utilisées pour émettre une ribambelle de certificats contrefaits. Enfin, ces autorités de certification dépendent toutes des législations des pays dans lesquels elles sont implantées. En France, par exemple, la récente Loi Renseignement prévoit que les "prestataires de cryptologie" - c'est à dire les autorités de certification - soient tenus de remettre dans les 72 heures leurs clés privées aux services de renseignement qui en feraient la demande.

## DNSSEC et DANE

La première et, probablement, la plus efficace des solutions qu'il est possible de mettre en place pour renforcer la sécurité de la PKIX s'appuie sur _DNSSEC_ (pour _Domain Name System Security Extensions_), une extension du système de résolution de noms _DNS_. _DNSSEC_ permet de signer cryptographiquement les composantes d'un nom de domaine : la racine du _DNS_, le _TLD_ (le _Top Level Domain_ `.info`, dans mon cas) et, enfin, les enregistrements de la zone _DNS_ du domaine (`nonblocking.info`, toujours dans mon cas). Les clés publiques correspondant aux clés privées utilisées pour les différentes signatures sont publiées afin que les résolveurs _DNS_ et autres clients puissent vérifier l'authenticité des enregistrements renvoyés par les serveurs _DNS_ faisant autorité.

L'idée de _DANE_ (pour _DNS-based Authentication of Named Entities_) est d'une certaine manière de "coupler" _TLS_ avec _DNSSEC_, d'utiliser l'infrastructure _DNS_ et _DNSSEC_ pour limiter les effets potentiels d'une attaque contre l'infrastructure à clé publique X.509 (en théorie, il serait même possible de se passer totalement de la _PKIX_ et d'utiliser _DNS_ comme seule infrastructure de distribution des clés publiques). Pour ce faire, l'opérateur du serveur _TLS_ publie, sous forme d'enregistrements _DNS_ signés cryptographiquement (les enregistrements _TLSA_), le certificat du serveur ou celui de son autorité de certification. Lorsqu'un client souhaite se connecter au serveur via _TLS_, il pourra comparer le certificat fourni par le serveur pendant le _handshake TLS_ aux informations qu'il a récupéré lors de la résolution _DNS_, s'assurant ainsi que le certificat semble bien légitime.

_DANE_ est une solution élégante, mais qui ne va pas sans contraintes. _There's no silver bullet_, comme disait je ne sais plus trop qui. La plus importante de ces contraintes, si vous souhaitez mettre en oeuvre _DANE_, c'est qu'il vous faudra gérer vos propres serveurs _DNS_ pour les zones concernées. Dans le premier cas, vous devrez non seulement administrer deux serveurs (un maître et un esclave), là où la plupart d'entre nous délègue habituellement la gestion de ses _DNS_. Il sera également impératif de maintenir vos zones _DNS_ et de gérer la rotation des signatures et des clés (les signatures devant être renouvelées tous les 30 jours et les clés plusieurs fois par an). Il est bien sûr possible d'automatiser ces tâches mais cette mise en oeuvre nécessitera une bonne connaissance du fonctionnement de _DNS_, du logiciel _DNS_ utilisé (_Bind_ par exemple) et de _DNSSEC_. L'alternative est de s'appuyer sur un fournisseur de services _DNS_ qui gèrera cela pour vous, mais cela signifie qu'il disposera de vos clés privées pour _DNSSEC_ (question de confiance, encore). Par ailleurs, si la plupart des résolveurs _DNS_ supportent _DNSSEC_, les navigateurs Web Firefox et Chrome ne supportent _DANE_ que par l'intermédiaire d'extensions.

Si vous êtes soucieux, et vous devriez probablement l'être, de protéger vos noms de domaines (et donc vos utilisateurs), _DNSSEC_ est la seule solution viable. _DANE_ n'apportant que peu de complexité par rapport à _DNSSEC_. Ainsi, si vous êtes prêt à faire l'effort de gérer vos propres serveurs _DNS_ pour _DNSSEC_, il n'y a aucune raison de ne pas en profiter pour mettre _DANE_ dans votre assiette. Il est également intéressant de noter que _DANE_ fonctionne avec d'autres protocoles que _HTTP_, par exemple _STARTTLS_ pour _SMTP_.

## Public Key Pinning

Là où _DANE_ s'appuie sur un second protocole, _DNS_, qu'il associe à _HTTP et à TLS_ pour vérifier l'authenticité des certificats, _HPKP_ (pour _HTTP Public Key Pinning_) s'appuie uniquement sur le protocole _HTTP_, plus particulièrement sur l'émission par le serveur d'un en-tête _HTTP_.

En effet, c'est le serveur _HTTP_ lui-même qui diffuse, via l'en-tête `Public-Key-Pins` une liste d'empreinte des clés publiques des certificats que le serveur souhaite voir "épingler". Le navigateur Web met en cache cette liste pour une durée déterminée par le serveur (la durée d'épinglage, si possible assez longue). A chaque nouvelle connexion, le client vérifiera que le certificat fourni par le serveur correspond aux informations qu'il avait mises en cache. Sauf à ce qu'il ne se soit emparé de la clé privée du serveur, un attaquant ne pourra se substituer à ce dernier puisqu'il sera incapable de fournir un certificat correspondant à l'un des éléments de la liste épinglée au préalable par le client. _HPKP_ met ainsi en oeuvre le principe _TOFU_ (pour _Trust On First Use_), utilisé également par _SSH_.

Le principal intérêt de cette méthode est qu'elle est très simple à mettre en oeuvre. La seule manipulation additionnelle par rapport à une configuration _TLS_ "classique", est de préparer des clés privées et publiques "de sauvegarde" à l'avance et de placer les empreintes correspondantes dans l'en-tête. Ces empreintes permettront d'assurer le renouvellement des certificats - ou de parer à la perte de la clé privée - dans de bonnes conditions. _HPKP_ est supporté par les versions récentes de Chrome et Firefox.

Le défaut d'_HPKP_ est que, s'appuyant sur le principe _TOFU_, cette technique ne permet pas de protéger d'attaques _MiTM_ les utilisateurs se connectant pour la première fois, en particulier pour des sites à faible trafic où les connexions sont souvent irrégulières.

## EOF

La _PKIX_ souffre de défauts qui impliquent qu'on ne peut malheureusement placer en elle qu'une confiance limitée. _DNSSEC_ et _DANE_ représentent à ce jour et à mon avis, l'option la plus efficace pour contrebalancer les faiblesse de la _PKIX_. En complément ou comme alternative à _DANE_, _HPKP_ est une solution relativement efficace et surtout très simple à mettre en oeuvre pour les serveurs web.

##### Notes

1. <a name="ca-validations"></a> La vérification la plus courante est celle que l'organisation requérant la signature d'un certificat soit bien titulaire du nom de domaine correspondant. Il existe aussi des procédures dites étendues (et plus chères) dans lesquelles le requérant doit fournir un certain nombre de documents administratifs visant à établir son existance légale.
2. <a name="pki-x509"></a> La principale _PKI_ est celle définie par X-509, il s'agit de l'ensemble des organisations et moyens techniques impliqués dans le processus de vérification de l'identité des différentes parties, de délivrance et de vérification des certificats.
3. <a name="afnic-dnssec-dane"></a> Pour une vision plus détaillée mais abordable de _DNSSEC_ et de _DANE_, l'AFNIC a [publié un dossier assez complet](https://www.afnic.fr/fr/produits-et-services/services/dnssec-16.html).

_Ce billet a été originalement publié sur [mon blog personnel](https://nonblocking.info/in-bed-with-tls-part-ii/)._
