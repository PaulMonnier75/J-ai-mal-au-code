---
layout: post
title: L'Actor Model avec AKKA .NET
description: TECH
image: assets/images/articles/akka/akka-logo.png
author: Paul MONNIER
---

<p>
    <span class="image right" style="max-width:32%;margin-top:-7%;"><img src="/assets/images/articles/akka/akka-logo.png" alt="" /></span>

    Dans ce post je parlerai de <b>AKKA .Net</b>, un <b>Framework</b> qui m'était encore totalement inconnu il y a quelques mois. 

    Akka .Net est un ensemble de librairies initialement venues de l'univers <b>Java</b>, adaptées en .Net par Petabridge. Ce Framework permet de créer facilement des systèmes <b>concurrents</b>, <b>distribués</b> et <b>scalables</b>, basé sur le pattern <b>"Actor Model"</b>.<br/><br/>
</p>


<h1 style="text-decoration: underline dotted;">Le pattern Actor Model</h1>

Ce pattern créé pour simplifier la gestion des <b>traitements parallèles</b>, est utilisé par Akka .NET et d'autres Frameworks comme Microsoft Orleans ou bien Proto.Actor, pour n'en citer que quelques uns.



<h3>Qu'est ce qu'un acteur Akka ?</h3>
<div>
    Un acteur est une entité chargée de traiter des messages reçus en entrée. Chaque acteur possède :
    <ul>
        <li><b>Une mailbox</b></li>
        <li><b>Un comportement appelé "behavior"</b></li>
        <li><b>Un state (optionnel)</b></li>
        <li><b>Des enfants (optionnel)</b></li>
    </ul>
</div>
<p>
    Je visualise un acteur comme un ouvrier qui <b>reçoit des instructions sous forme de messages dans une "mailbox"</b>. La boite aux lettres est une <b>"Queue"</b> dans laquelle les messages sont empilés et dépilés  en <b>FIFO</b> (First In First Out), <b>le premier message arrivé est le premier à être traité</b>. Un acteur peut également créer et communiquer par messages avec d'autres acteurs ou bien avec ses enfants sans se soucier de leur état, ni de leur localisation.<br/><br/>

    <b>Les messages sont traités de manière séquentielle</b>. Attention cela ne veut pas dire que le programme sera synchrone. Evidemment, les acteurs peuvent travailler simultanément puisqu'ils sont totalement isolés les uns des autres. Disons que vous ayez 3 messages à traiter simultanément, il vous suffit de créer 3 acteurs et d'envoyer ces 3 messages à chacun d'entre eux, les processus s'exécuteront en parallèle.<br/><br/>

    Lorsqu' un message est dépilé, l'acteur a la possibilité d'effectuer de multiples opérations. Il peut appliquer un <i>'behavior'</i> qui lui permettra de modifier son comportement appliqué aux messages. Si l'acteur n'est pas <i>'stateless'</i> il peut alors appliquer des opérations sur son <i>'state'</i> qui est encapsulé et isolé des autres acteurs. Un acteur a également la faculté de se rappeler lui-même ou bien de créer des acteurs enfants auxquels il pourra déléguer des tâches.<br/><br/>

    <b>Il est important de comprendre que les acteurs sont totalement isolés les uns des autres, ils n'ont donc pas de mémoire partagée et ne pourront en aucun cas modifier directement le state d'un autre acteur.</b>
    
    <span class="image right" style="max-width:17%;"><img src="/assets/images/tips.png" alt="" /></span>

    <br/>
</p>

<div>
    <b>Conseils :</b>
    <ul>
        <li>Il est fortement conseillé d'utiliser des <b>messages immutables</b> pour éviter tout anti-pattern</li>
        <li>Vous pouvez <b><i>'override'</i></b> la mailbox de vos acteurs pour modifier leurs comportements. Par exemple vous pourriez faire une <b><i>'Priority Mailbox'</i></b>  chargée de dépiler vos messages en les priorisant grâce à un attribut placé dans le model de vos messages.</li>
    </ul>
</div>



<h3>La Clusterisation des acteurs</h3>

<p>
    <span class="image left" style="max-width:35%;"><img src="/assets/images/articles/akka/clusterisation-akka.jpg" alt="" /></span>
    
    <br/><br>
    Un cluster est <b>un rassemblement de machines qui agissent comme un seul et même système</b>, permettant de mieux <b>répartir les charges de travail</b> et des <b>paralléliser des taches</b> sur plusieurs serveurs.<br/>
    L'un des gros points forts de ce Framework est qu'il vous permettra de <i>clusteriser</i> facilement vos acteurs sur différents nœuds / serveurs d'un système.<br/><br/><br/>
</p><br><br>


<h3>Comment communiquer avec un acteur ?</h3>

<div>
Il existe 3 moyens différents pour envoyer un message à un acteur :<br/><br/>
    <p>
        <span class="image left" target="_blank" style="max-width:30%;"><img src="/assets/images/articles/akka/tell.jpg" alt="" /></span><br>
        <b style="font-size:28pt;color:#ff4500;">Tell :</b> Permet simplement d'envoyer un message à un acteur sans le bloquer. C'est une sorte de <b>"Fire And Forget"</b>
    </p><br/>
    <p>
        <span class="image right" target="_blank" style="max-width:30%;"><img src="/assets/images/articles/akka/ask.jpg" alt="" /></span>
        <b style="font-size:28pt;color:#ff4500;">Ask :</b> Contrairement à Tell, cette méthode est <b>bloquante</b> dans la grande majorité des cas. En effet, si vous décidez d'attendre la fin de la Task, l'acteur sera bloqué jusqu'à la réception de la réponse de son enfant. <b>Cette réponse remontera au fur et à mesure le flux d'appel</b>
    </p><br/>              
    <p>
        <span class="image left" target="_blank" style="max-width:30%;"><img src="/assets/images/articles/akka/forward.jpg" alt="" /></span><br>
        <b style="font-size:28pt;color:#ff4500;">Forward :</b> Ressemble beaucoup à Tell à contrario que l'acteur 4 n'as pas besoin de faire passer la réponse aux acteurs 1 et 2, il peut <b>directement répondre à l'acteur 1</b>. Contrairement à Tell lorsque Forward est utilisé, la référence du Sender passé à l'acteur 4 est celle de l'acteur 1, alors qu'avec Tell la référence aurait été celle de l'acteur 3
    </p>
</div><br>

<span class="image right" style="max-width:14%;"><img src="/assets/images/warning.png" alt="" /></span>

<b style="font-size:28pt">Attention</b>, <b>la grosse différence entre Ask et Tell n'est pas la réponse attendue mais bel et bien le fait que votre acteur soit paralysé ou non.</b> 
Lorsqu'un de vos acteurs utilise la méthode <b>Ask, il est dans l'incapacité de dépiler et de traiter un nouveau message tant qu'il n'aura pas reçu une réponse de son enfant</b> (Si vous décidez d'await votre Task).
Vous ne comprenez peut être pas le rapport avec la méthode Tell puisque comme je l'ai dit juste au dessus cette méthode permet uniquement d'envoyer un message en Fire And Forget. Il est vrai que lorsqu'un message est envoyé à un enfant à l'aide de la méthode Tell, l'acteur ne se préoccupe plus du message envoyé et peut passer au suivant sans se soucier d'une potentielle réponse. 
Cependant, sachez que <b>tous les acteurs ont une référence vers leur Sender</b>. Si vous souhaitez que l'acteur ayant reçu un message par la méthode Tell réponde à son parent, vous pouvez utiliser Sender.Tell, cependant sachez que le parent n'aura pas attendu votre réponse et aura probablement fait d'autres actions.<br><br>


<h3>La gestion d'erreurs avec AKKA .Net</h3>
<h4>Le cycle de vie</h4>

Avant de parler de la gestion d'erreur, il est important de connaitre le cycle de vie d'un acteur.<br>
Comme vous pouvez le voir sur l'image ci dessous, il y a plusieurs étapes entre l'instanciation d'un acteur et son arrêt.<br>

<span class="image left" target="_blank"  style="max-width:41%;">
    <img src="/assets/images/articles/akka/life-cycle.jpg" alt="" />
</span>

1. La première phase est le démarrage <b><i>'Starting'</i></b>, durant cette étape l'acteur est initialisé dans l' 'Actor System'.<br>
2. Une fois avoir été initialisé, l'acteur rentre dans l'état <b><i>'Receiving Messages'</i></b>. Il reçoit, dépile et traite les messages durant cette phase.<br>
3. Une demande d'arrêt ou bien un acteur parent qui est en exception peut mettre un acteur dans l'état <b><i>'Stopping'</i></b> durant lequel il effectuera le nettoyage de son state interne.<br>
4. Si l'acteur rentre dans l'état <b><i>'Terminated'</i></b> alors cela veut dire que l'acteur est complètement mort, il ne recevra donc plus de messages et ne pourra plus être redémarré.<br>
5. Si l'acteur parent à constaté une erreur il peut alors demander à son enfant de redémarrer et rentrera donc dans le dernier état qui est le <b><i>'Restarting'</i></b>, l'ultime état avant que l'acteur ne reprenne son état initial de 'Starting'<br>

Entre tous ces états, quelques méthodes du cycle vie sont appelées, il est possible de les <i>'override'</i> pour leur donner un comportement spécifique.

<table>
    <colgroup>
        <col width="20%" />
        <col width="80%" />
    </colgroup>
    <tbody>
        <tr>
            <td markdown="span">**PreStart()**</td>
            <td markdown="span">Appelée avant la réception du tout premier message, vous pouvez y ajouter du code spécifique comme par exemple y ouvrir le fichier dans lequel votre acteur devra écrire.</td>
        </tr>
        <tr>
            <td markdown="span">**PostStop()**</td>
            <td markdown="span">Appelée lorsque l'acteur a été arrêté et ne reçoit donc plus de messages. Vous pouvez libérer les ressources dont vous ne vous servirez plus ici.</td>
        </tr>
         <tr>
            <td markdown="span">**PreRestart()**</td>
            <td markdown="span">Appelée avant que l'acteur ne soit redémarré, vous pouvez donc sauvegarder le dernier message qui n'a pas pu être traité ou bien "logger" l'exception qui a causé l'arrêt de votre acteur.</td>
        </tr>
         <tr>
            <td markdown="span">**PostRestart()**</td>
            <td markdown="span">Appelée juste après le PreRestart() et juste avant le PreStart().</td>
        </tr>
    </tbody>
</table>

Lorsqu'un acteur reçoit un message d'arrêt alors il va : 
1. <b>Demander à ses acteurs enfants de s'arrêter, ses acteurs enfants demanderont à leurs enfants de s'arrêter et ainsi de suite jusqu'à avoir atteint le bas de la hiérarchie.</b>
2. <b>Finir d'exécuter le message en cours de traitement.</b>
3. <b>Placer les autres messages présents de la mailbox à un endroit appelé les Dead Letters de l' Actor System, ils ne seront donc pas traités par l'acteur.</b>

L'arrêt d'un acteur s'effectue de manière <b>asynchrone</b>, néanmoins vous pouvez utiliser la méthode <i>"GracefulStop(TimeSpan)"</i> qui permet de await l'arrêt de votre acteur. Le TimeSpan passé en paramètre est un timeout à partir duquel une exception sera levée en cas de dépassement de cette limite.

<h4>Les decisions</h4>

Prenons pour exemple le graphique ci-dessus représentant une hiérarchie d'acteurs. 
Que se passe t'il si l'acteur 2 lève une exception ? <br>
Dans un premier temps les acteurs 4 et 5 seront suspendus, puis l'exception sera remontée au parent (Acteur 1) qui aura le choix d'appliquer l'une des décisions ci-dessous:

<span class="image right" target="_blank"  style="max-width:40%;">
    <img src="/assets/images/articles/akka/decisions.jpg" alt="" />
</span>

1. <b>Resume:</b> L'acteur 2 continue son exécution sans se soucier de l'exception levée, l'acteur garde son state, ses enfants ne sont plus suspendus et reprennent leur exécution.
2. <b>Restart:</b> L'acteur 2 et ses enfants redémarrent et perdent leur state, cependant les mailboxs sont sauvegardées. Une fois redémarré, l'acteur 2 et ses enfants reprennent le traitement de leurs messages.
3. <b>Stop:</b> L'acteur 2 et ses enfants sont arrêtés.
4. <b>Escalate:</b> L'acteur 1 remonte l'exception à se hiérarchie.

Imaginons que l'acteur 1 prenne la décision de redémarrer l'acteur 2, alors celui-ci sera redémarré ainsi que ses enfants (acteur 4 et acteur 5).<br>
 <b>Ils perdront donc leur state mais une fois redémarrés</b>, ils reprendront le traitement des messages présents dans leurs mailboxs respectives.

<h3>Supervision Strategy</h3>

<b>Les acteurs parents sont responsables des erreurs de leurs enfants.</b> Ils doivent donc appliquer ce que l'on appelle une <i>"Supervision Strategy"</i> pour déterminer ce qui va se passer lorsque l'un d'eux lèvera une exception.<br>
Si vous n'avez pas overridé de stratégie, les acteurs parents utiliseront la stratégie par défaut, autrement, vous pouvez personnaliser les décisions prises par votre acteur sur les deux types de stratégie ci-dessous..<br>

<ul>
    <li>La <b><i>"One for one strategy"</i></b> est la stratégie utilisée par défaut, elle appliquera une des décisions ci-dessus à l'acteur ayant levé l'exception.</li>
    <li>La <b><i>"All for one strategy"</i></b> appliquera une décision à tous ses enfants. Si l'acteur 1 décide de redémarrer l'acteur 2 alors l'acteur 3 sera également redémarré.</li>
</ul><br>






<h1 style="text-decoration: underline dotted;"><a href="https://github.com/PaulMonnier75/AkkaPOC">Cas Pratique</a></h1>

<p style="font-size:11pt;"><i>(Les images de code sont cliquables et ouvriront un nouvel onglet Github sur la partie de code en question)</i></p>

<p>
    Prenons une maison connectée, dans laquelle il y a des ampoules connectées, un thermostat connecté et une Chromecast. L'application une API permettant de gérer les commandes de nos différents appareils connectés.<br/>
    Dans cet exemple l'utilisation de Akka .NET n'a pas une grande utilité car le principe même de Akka .Net est de pouvoir ingurgiter des millions de commandes par seconde et cela n'est pas le cas de cette application, néanmoins je trouve que c'est un bon exemple pour illustrer et expliquer les grandes lignes de Akka.<br/>

<div style="display:flex;justify-content:center">
    <span class="image" style="max-width:90%;"><img src="/assets/images/articles/akka/architecture-maison-connectee.jpg" alt="" /></span>
</div>

<div style="display:flex;justify-content:center">
    <p style="font-size:11pt;"><i>Ci dessus l'architecture de l'application que nous allons concevoir</i></p>
</div>

En premier lieu l'API reçoit une requête HTTP, prenons pour exemple une requête POST sur la route "setHomeTemperature", avec le paramètre 20.5, correspondant à la température désirée. Une fois la requête reçue nous allons créer un objet de type <b>SetTemperatureCommand</b> qui hérite d'une classe abstraite <b>HomeAutomationCommand</b> qui elle-même hérite d'une autre classe abstraite nommée <b>Command</b>.<br/><br/>

<a class="image left" target="_blank" style="max-width:40%;" href="https://github.com/PaulMonnier75/AkkaPOC/blob/master/Core/Models/Command.cs">
    <img src="/assets/images/articles/akka/setTemperatureCommand.jpg" alt="" />
</a>

La commande SetTempartureCommand possède un attribut contenant la valeur voulu que l'on set à la création de l'objet.<br/><br/>
Les objets de type <b>"Command"</b> que nous venons de voir ci-dessus, <b>représentent les messages immutables qu'utilisent nos acteurs pour communiquer entre eux.</b><br/><br/><br/>

<a class="image right" target="_blank" style="max-width:40%;" href="https://github.com/PaulMonnier75/AkkaPOC/blob/master/Core/Core.cs">
    <img src="/assets/images/articles/akka/handleCommand.jpg" alt="" />
</a>

<b>Puisque toutes nos commandes héritent de la même classe abstraite Command nous pouvons unifier le point d'entrée avec la méthode HandleCommand.</b><br/>
Une fois arrivé dans cette méthode je fais appel à mon premier acteur, en utilisant la méthode Tell qui prend en argument la la commande souhaitée.
</p>

Voyons maintenant comment l'acteur principal (HandleCommandActor) gère ses messages.<br><br>
<a class="image" target="_blank"  style="max-width:100%;" href="https://github.com/PaulMonnier75/AkkaPOC/blob/master/Core/Actors/CommandHandlerActor.cs">
    <img src="/assets/images/articles/akka/command-handle-actor.jpg" alt="" />
</a><br>

Avant tout, rappelez-vous que  SetTemperatureCommand hérite de Command mais également HomeAutomationCommand, ce qui permet de bien séparer les appareils.<br>
<b>Comme vous pouvez le voir dans le constructeur, un filtre est fait sur le type de la commande reçue.</b> La méthode Receive<T> est propre au ReceiveActor dont hérite mon CommandHandlerActor et permet de rediriger la commande vers une méthode.

<span class="image left" target="_blank"  style="max-width:38%;">
    <img src="/assets/images/articles/akka/pattern-matching.png" alt="" />
</span>

<br><br>
Si cette syntaxe ne vous plaît pas, depuis C# 7, vous pouvez utiliser le pattern matching du switch pour filtrer vos messages.
<br><br>

<div>
    <b>Rappel:</b>
    <ul>
        <li>Premier type d'abstraction: Command</li>
        <li>Second type d'abstraction: HomeAutomationCommand</li>
        <li>Troisième type d'abstraction: SetTemperatureCommand</li>
    </ul>
</div>

<b>Si vous avez toujours le schéma de l'application en tête, vous devriez vous souvenir que mon acteur principal délègue la commande reçue à ses enfants. Dans l'exemple de SetTemperatureCommand, cette dernière est déléguée à HomeAutomationActor chargé des commandes du type HomeAutomationCommand (second type d'abstraction) qui déléguera également sa tâche à un autre acteur enfant pour toutes les commandes du troisième type d'abstraction.</b><br>

<a class="image right" target="_blank"  style="max-width:55%;" href="https://github.com/PaulMonnier75/AkkaPOC/blob/master/Core/Actors/ThermostatActor.cs">
    <img src="/assets/images/articles/akka/thermostat-actor.jpg" alt="" />
</a>

Pourquoi déléguer toutes ces tâches à des acteurs enfants ? <br>
Nous pourrions imaginer que l'acteur principal puisse s'occuper de faire tout le travail, cependant la lisibilité et la propreté du code seraient impactées. Il est donc plus judicieux de suivre le principe de <b>Single Responsibility Principle</b> (un des principe SOLID) pour que chaque entité s’occupe d’un seul et même périmètre d’actions.<br>
C'est pourquoi dans mon exemple, le ThermostatActor s'occupe uniquement des actions propres au thermostat, notamment avec la méthode "ConvertFahrenheitToCelcius" qui se charge de convertir la température reçue et qui sera ensuite envoyée à l'API du thermostat.<br>

<a class="image left" target="_blank"  style="max-width:54%;" href="https://github.com/PaulMonnier75/AkkaPOC/blob/master/Core/Actors/HomeAutomationActor.cs">
    <img src="/assets/images/articles/akka/supervision-strategy.png" alt="" />
</a><br>

Ci-contre, un exemple montrant comment j'ai overridé la <i>'Supervision Stratégie'</i> de l'acteur HomeAutomationActor. Vous remarquerez que  j'ai décidé d'<i>'override'</i> la <i>OneForOneStrategy</i>, signifiant que si mon ThermostatActor lève une exception il sera le seul à recevoir une décision, le LightActor ne sera donc pas impacté.<br>

En exécutant la requête suivante le ThermostatActor lèvera une exception de type ElectricityOverConsommationException. Suite à la surcharge de la stratégie, l'acteur parent va ordonner à son enfant de redémarrer. J'ai pris le soin de placer des <i>'Console.WriteLine'</i> dans les méthodes du cycle de vie pour mieux comprendre le fonctionnement.<br><br><br><br>

<span class="image" target="_blank"  style="max-width:100%;">
    <img src="/assets/images/articles/akka/postman-sample.png" alt="" />
</span>

<h2>Conclusion</h2>

A travers cet article, nous avons vu les grandes lignes du pattern acteur model avec Akka. J'espère que cela vous aidera à démystifier ce concept qui n'est pas forcément évident à comprendre.<br>
J'aime beaucoup ce Framework puisqu'il permet de créer facilement des applications distribuées, concurrentes et scalables à travers de très bonnes APIs.<br>
De plus la <i>'Supervision Strategy'</i>, permet à votre application d'être résiliente en lui permettant de continuer de fonctionner en cas d'erreurs. 
Le framework est très vaste et nous avons vu qu'une infime partie de ce que Akka peut nous donner.
J'ai la chance de travailler sur un projet sur lequel Akka est mis en place, nos acteurs traitent des millions de commandes asynchrones par jour. Nous avons overridé nos mailbox pour que nos commandes aient une notion de priorité et que certains messages soient traités avant d'autres.<br><br>

<p>
    <span class="image left" style="max-width:45%;"><img src="/assets/images/articles/akka/proto-actor-benchmark.png" alt="" /></span>

    Il semblerait que <b>Proto.Actor</b> soit le nouveau Framework visant à remplacer ce Akka.<br/>
    Développé par le papa de Akka, il serait plus performant avec de nouvelles fonctionnalités.

    <b>Roger JOHANSSON</b> a voulu recommencer un Framework d' Actor Model sur une base saine, permettant ainsi de rajouter des fonctionnalités qui auraient été couteuses et compliquées à développer sur son ancien Framework.<br/>

    Bien que cette technologie soit prometteuse et plus performante, le manque de documentation m'a pour le moment découragé à aller plus loin dans mon POC.<br/>

    <p style="font-size:11pt;">Vous trouverez ci-contre un Benchmark comparatif pour votre curiosité.</p> 
</p>

 <a href="https://github.com/PaulMonnier75/AkkaPOC">Lien Github</a>