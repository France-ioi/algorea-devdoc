---
layout: page
title: Drafts
nav_order: 999
parent: forum
---

This is a summary of discussions (in French) we had in January 2023. 
These have been either translated to design decisions or left for later. They are just pasted here for the records, it is not needed to read this page.

~~~


1- Sous quelle(s) condition(s) un utilisateur peut-il voir son propre thread ?

    propos 1.1:  si il peut demander de l'aide (= il ne voit pas l'historique si il perd les droits de demander de l'aide!) 


2- Sous quelle(s) condition(s) un utilisateur peut-il demander de l'aide ? (= activer "aide demandé" sur son thread)

    propos. 2.1: si il a un reçu un droit spécifique "can_request_help_to" pour cet item 


3- Qui peut écrire dans un thread ?

    propos 3.1:

    1. le participant concerné par le thread, OU

    2. un utilisateur qui a le droit can_watch>=answer sur l'item ET can_watch sur le participant OU

    3. si

    - "demande d'aide" active ET

    - l'utilisateur fait partie du groupe autorisé pour la "demande d'aide" ET

    - a un accès can_watch>=answer sur l'item OU a validé l'item

[discussions 25/01/23]  dans le cas 1 et 2, écrire dans un thread ouvre le thread

4- Qui peut lire un thread?

    propos 4.1: 

    ceux cités dans propos 3.1

2. tout utilisateur qui a interagi avec le thread (càd qui a posé une action? peut être révoqué?)
M: Pour le 4.1 je ne suis pas du tout sur que le 2. soit une bonne idée. Ça me semble poser plus de problèmes qu'autre chose.
D: Je pense que c'est une demande qui vient de toi à l'origine (pas d'hier mais précédemment)
Sinon, ça veut dire que si l'élève active l'aide, un aidant répond, puis 1h plus tard l'élève répond un truc puis désactive l'aide... l'aidant ne peut pas voir la réponse, ni si l'utilisateur a réussi l'exo... le thread disparait juste de sa visibilité.
M: il me semble que c'est une question différente : qui peut continuer à voir une thread une fois que la thread est clôturée. Et du coup ça pose la question de ce que veut dire clôturé. On peut voir ça de 2 manières :
    A. on a eu la réponse à sa question et du coup n'a plus besoin que d'autres viennent aider, ce qui pourrait se faire manuellement, ou automatiquement selon des critères comme le fait d'avoir résolu le sujet, le fait de ne pas avoir réagi pendant x jours après le dernier message. 
    B. on ne veut plus que personne puisse voir l'échange, pour une raison ou une autre (par exemple on a dit un truc dont on n'est pas fier).
Dans le cas A. les personnes qui ont aidé peuvent avoir envie de voir l'éventuel "merci" qui correspond à ton cas. Des personnes qui n'ont pas aidé mais avaient accès récemment au contenu peuvent aussi voir les threads récemment clôturées. Mais du coup ça reste couvert par la proposition 1
La proposition 2 ce serait pour le cas où on n'avait le droit d'aider mais tout d'un coup on le perd, par exemple du fait de la situation B. Dans ce cas j'ai plutôt l'impression qu'il faut définitivement bloquer l'accès.
En gros, je ne vois pas de cas où il faudrait bloquer l'accès à ceux qui pouvaient voir la thread avant mais n'ont pas aidé, mais pas la bloquer à ceux qui ont aidé.
D: " qui peut continuer à voir une thread une fois que la thread est clôturée" ->  un thread n'est jamais vraiment cloturé définitivement vu qu'on peut vouloir reposer une question plus tard (un thread étant identifié par un item-participat). C'est pour ça que je considérais un flag "demande d'aide" activé ou pas sur un couple participant-item. Donc, ce que tu veux dire c'est que qd on "désactive la demande d'aide", on garde la date-heure de cette désactivation et on laisse en fait l'accès x jours à ceux qui y avaient accès (x étant fixé par nous pour toute la plateforme)... et qu'il y aurait un mode (où?) pour couper l'accès direct ?
[discussions 25/01/23]   
Il faudrait 3 status pour un thread: 
    fermé, 
    ouvert en attente de réponse de l'utilisateur, 
    ouvert en attente de réponse d'un aidant
Lorsqu'un thread est cloturé, l'utilisateur et les personnes avec le droit d'observation auraient tjr le droit de voir le thread. Les autres pourraient voir un thread cloturé pendant 2 semaines... et ceux avec des droits de watch sur le sujet pourraient tjr garder cet accès.

5- Qui peut donner le droit "can_request_help_to"?

    propos 5.1: can_grant_view >= enter

    propos 5.2: can_grant_view >= content 


6- Comment se propage le droit "can_request_help_to" à travers les items?

    propos 6.1: propagé si content_view_propagation >= as_info

M: Pour le 6. je n'ai plus en tête les différents modes. On a ça dans l'interface qq part d'ailleurs ?
D: Ben oui, ce sont  les "hidden+locked" (none), "locked" (as_info), "open" (as_content)  de la propagation. Le reste de la propagation n'est pas dans l'interface (ça va d'ailleurs être bloquant un jour).
[discussions 25/01/23]  On ne peut pas utiliser la propagation de can_view pour le droit "can_request_help_on" pcq un créateur de contenu pourrait choisir de ne pas propager du tout "view" entre un chapitre et son contenu pour que le contenu apparaisse avec un déblocage, mais tt de mm autoriser la propagation de "can_request_help_on" dans ce cas. Donc il faut une propagation spécifique. Une autre option serait d'avoir un autre niveau de propagation entre "none" et "as_info" mais cela semble plus compliqué.

7- Quel(s) valeurs(s) pour le droit "can_request_help_to"?

    Propos 7.1.: group_managers, group_members, all - 

    Problème: ça demande de savoir à qui a été donné ce droit, complique la façon dont les droits sont récupérés de façon aggrégés, un utilisateur peut d'ailleurs avoir plusieurs fois ce droit selon plusieurs groupes différents (les permissions sont rapportées dans énormément de services utilisés très fréquemment)

    Problème: les managers d'un groupe n'étant pas un groupe, la requête fréquente "quelles sont les 'aides demandées' récentes qui me sont destinées devient un peu compliquée

    Propos 7.2:  group_id OR NULL (=all) 

    Q: Quels groupes peuvent être proposés à la personne qui met les droits ? 

    ceux qui sont visible par le group entier, càd le groupe, ses descendants, ses ancêtres, ses managers (direct ou des ancêtres)

    que fait-on si un manager disparait du groupe ou si un groupe ancêtre ne l'est plus?

M: effectivement 7.2 semble plus simple tout en étant aussi plus souple je trouve. Pour la question, je dirais qu'il faudrait que les managers d'un groupe puissent voir qui a  obtenu le droit de demander de l'aide à ce groupe, et du coup potentiellement le retirer.

8- Qu'est-ce que le participant peut choisir comme groupe à qui demander de l'aide?

    Propos 8.1: 

    selon le groupe donné dans la permission "can_request_help_to" (possiblement "all")

    Si pas "all": tous les sous-groupes de ce groupe

    Si "all":  n'importe quel groupe/utilisateur visible par l'utilisateur (???)

M: Pour 8.1, le 3e point si "all", je ne sais pas trop. On est d'accord que si je demande de l'aide à une personne, les gens de 3.1.1 et 3.1.2 pourront aussi voir cette thread donc que ça ne permet pas aux élèves de discuter entre eux en se "cachant" de leur prof.
Par ailleurs si on demande de l'aide à une personne précise, ça n'attire pas plus l'attention de cette personne que si on demande de l'aide à tout le monde.
(Je me demande si on pourra un jour tagguer des gens pour attirer leur attention comme sur discord)
Si on peut choisir n'importe-qui, on peut facilement se retrouver à demander de l'aide à quelqu'un qui ne peut pas voir la thread. Dans ce cas ça impliquerait peut-être un avertissement, sauf que ça donnerait du coup un moyen de tester si quelqu'un a résolu ou non un exo, À moins que l'avertissement soit générique : "Ce groupe / cette personne ne pourra vous aider que si ..."
Bref du coup en faisant attention à ça, je ne vois pas de véritable raison d'empêcher quelqu'un de choisir un groupe ou utilisateur quelconque, à part qu'il faut s'assurer que l'utilisateur comprend ce que ça implique.

9- Un thread est-il unique pour un (item,participant) ? (participant=utilisateur ou team) 

    OUI Implique que si il fait une demande d'aide sur un item seulement aux managers, puis 1an plus tard sur le mm item à tous les utilisateurs, tous les utilisateurs pourront voir sa discussion avec les instructeurs de l'année précédente? 

    ce qui implique qu'il peut "partager" aussi volontairement des choses que des instructeurs lui auraient dit (que les managers ne voulaient pas fuiter?)

    NON implique toute une gestion (compliquée?) dans l'UI pour pouvoir créer un nouveau thread, parcourir les anciens threads, ... pour l'utilisateur et les instructeurs; créer un nouveau...

M: Pour 9. dans mon idée ça a toujours été unique pour un (item, participant)
Pour le coup des fuites, j'ai envisagé plusieurs pistes, mais le plus simple me semble être de prévenir que l'ensemble des gens qui peuvent voir les messages peut être amené à s'élargir à tout moment, donc qu'il ne faut pas considérer que c'est un message confidentiel.
De toutes façons rien n'empêche de diffuser une capture d'écran.
D: En fait la question "unique pour un (item,participant)" est aussi "ne serait-ce pas (item, participant, attempt)?". Si c'est (item,participant), ça implique que si un utilisateur à 2 attempts ouverts en mm temps (2 languages), je ne sais pas vers quelle page (best answer) renvoyer le prof.
[discussions 25/01/23] L'attempt peut être nécessaire pour savoir vers quelle version du pb l'utilisateur pose la question, en effet. Mais par contre, il ne faut pas être bloqué sur un attempt et avoir des difficultés à voir les autres questions (probablement liées) sur l'attempt précédent ou suivant.
La solution retenue est de stocker l'"answer" complète liée à chaque message dans dyndb (id nécessiterait de s'assurer qu'il y a bien une sauvegarde à chaque message)

10- Que veut-on comme info (dans un premier temps) dans la liste des threads ? (et par quoi trier)
Infos:
    - demande d'aide activée (datetime?) ou pas
    - dernière réponse de l'utilisateur ou d'un autre utilisateur
[discussions 25/01/23] :
    Le titre de l'item (et son père -> à faire via l'overlay habituel)
    l'utilisateur
    status du thread
    nb de msg (quitte à résumer à >100 qd il y a bcp de messages)
    date dernier msg
    score (de l'attempt si plusieurs attempts idéalement)

[Discussions 25/01/23] Thread lu/non lu/suivi:
L'idée serait ne pas retenir pour chaque utilisateur son "état" par rapport à chaque thread qu'il aurait vu... en tout cas pas en SQL. 
Si il a participé à un thread (ou qu'il souhaite suivre un thread) et qu'il doit recevoir des notifications, cela peut être stocké dans DynamoDB. 
Pour l'instant, pour savoir ce qu'il faut afficher comme "à voir", on pourrait juste stocker la date-heure de la dernière fois que l'utilisateur a regardé la liste des threads (+ éventuellement ajouter un action manuelle pour ajuster cette valeur)

[Discussions 25/01/23] Modération
Pt de vue modération: il faudrait proabablement que les utilisateurs puissent éditer et supprimer un de leur message, mais on doit nous pouvoir retrouver une trace en cas de pb. Il devrait être possible de signaler un contenu comme problématique, dans ce cas là, le contenu serait caché de tous. Les signalements faux seraient sanctionnés.

----

Pb: le droit "can_request_help_to a group" n'est pas propageable à cause de son argument `group` qui n'est pas aggrégeable en une seule entrée (tout le système de permission serait à revoir)

Si l'utilisateur a ses droits de can_request_help retiré, lui coupe-t-on accès lecture/écriture aux threads ouverts?
- propos. 1 - oui: ça peut permettre à un prof de poser une question mm si l'utilisateur ne peut pas initier la conversation lui-mm
- propos. 2 - non: ça permet de considérer que retirer les droits coupe toute accès au forum immédiatement (par ex pour une question mise dans un concours)

Si l'utilisateur n'a de droit de can_request_help
, peut-il voir (read-only) un ancien thread fermé?
- propos 1 - oui: dans le cas d'un thread ouvert par un prof par ex, ça lui permet de voir la conclusion. Ou juste de voir ce qui a été discuté sans demander plus.
- propos 2 - non, c'est la façon de lui retirer accès
- propos 3 - oui, pdt 15 jours après la fermeture

Qui peut fermer un thread? 
- propos 1: l'initiateur (il faut retenir qui c'est)
- propos 2: tous ceux qui peuvent y écrire

Quel permission est requise (sur l'item) pour mettre le droit can_request_help_to ?