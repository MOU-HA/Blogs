---
ID: 74
post_title: >
  Gérer proprement les interruptions de
  threads en Java
author: ocroisier
post_date: 2009-06-09 13:58:00
post_excerpt: |
  <p>Voici la traduction rapide d'<a href="http://tech.puredanger.com/2009/06/08/interrupt-handling/">un article d'Alex Miller</a> (avec son aimable permission naturellement), qui récapitule les grands principes de gestion de l'interruption des threads.<br />
  Un sujet souvent mal maîtrisé, qui pourtant, vous allez le voir, est relativement simple à comprendre.</p>
layout: post
permalink: http://blog.zenika-offres.com/?p=74
published: true
---
<p>Voici la traduction rapide d'<a href="http://tech.puredanger.com/2009/06/08/interrupt-handling/">un article d'Alex Miller</a> (avec son aimable permission naturellement), qui récapitule les grands principes de gestion de l'interruption des threads.<br />
Un sujet souvent mal maîtrisé, qui pourtant, vous allez le voir, est relativement simple à comprendre.</p>
<!--more-->
<h4>Gestion des interruptions, par Alex Miller</h4> <p>Un thread Java ne peut pas être interrompu de manière préemptive. En revanche, une application peut appeler sa méthode <code>Thread.interrupt()</code>. Si le thread est en attente d'une opération bloquante à ce moment-là (comme <code>wait</code>, <code>join</code> ou <code>sleep</code>), une <code>InterruptedException</code> est levée. Dans le cas contraire, seul le flag <code>interrupted</code> du thread est activé.</p> <p>Un thread recevant une <code>InterruptedException</code> a plusieurs options&nbsp;:</p> <ul> <li><strong>Gérer lui-même l'interruption</strong>. Le traitement typique consiste à vérifier si une condition particulière a été atteinte (si le thread doit s'arrêter ou changer d'état), et à modifier le comportement du thread en conséquence.</li> <li><strong>Propager l'exception</strong>. Celle-ci est immédiatement relancée, et sa gestion incombe alors à du code de plus haut niveau.</li> <li><strong>Retarder la gestion de l'exception</strong>. Il suffit pour cela d'appeler <code>Thread.currentThread().interrupt()</code> de manière à réactiver le flag d'interruption du thread. Du code de plus haut niveau peut alors vérifier l'état de ce flag et prendre les mesures adéquates.</li> <li><strong>Ignorer l'exception</strong>. L'exception est capturée et étouffée silencieusement. Cette solution est à proscrire - à moins naturellement que le bloc <code>catch</code> ne contienne tout le code nécessaire au traitement de l'interruption,auquel cas il ne s'agit que d'un cas particulier de la première option.</li> </ul> <p>Réfléchissez soigneusement à la politique de gestion des interruptions de vos threads, et à leur réaction dans le cas où ils sont interrompus en pleine attente d'une opération bloquante. De manière générale, les frameworks techniques devraient propager les exceptions de manière transparente, et laisser les applications les gérer en accord avec la politique d'interruption de leurs threads.</p> <h4>En pratique</h4> <p>Voici un exemple de code gérant correctement l'interruption.<br />
Le thread lancé effectue des boucles d'une seconde à l'aide de la méthode bloquante sleep(). L'utilisateur peut déclencher l'interruption de ce thread en saisissant quelquechose dans la console, et constater que la boucle s'est terminée prématurément.</p> <pre class="java code java" style="font-family:inherit"><ol><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">class</span> ThreadInterruption</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #000000;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">static</span> <span style="color: #7F0055; font-weight: bold;">void</span> main<span style="color: #000000;">&#40;</span><span style="color: #000000;">String</span><span style="color: #000000;">&#91;</span><span style="color: #000000;">&#93;</span> args<span style="color: #000000;">&#41;</span> <span style="color: #7F0055; font-weight: bold;">throws</span> <span style="color: #000000;">Exception</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #000000;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #808080; font-style: italic;">// Démarrage du thread</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		CounterThread counter = <span style="color: #7F0055; font-weight: bold;">new</span> CounterThread<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		counter.<span style="color: #000000;">start</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #808080; font-style: italic;">// Attente d'une interruption manuelle</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #000000;">System</span>.<span style="color: #000000;">in</span>.<span style="color: #000000;">read</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #000000;">System</span>.<span style="color: #000000;">out</span>.<span style="color: #000000;">println</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;Interruption !&quot;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		counter.<span style="color: #000000;">interrupt</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #808080; font-style: italic;">// Attente de la fin du thread</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		counter.<span style="color: #000000;">join</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #000000;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">static</span> <span style="color: #7F0055; font-weight: bold;">class</span> CounterThread <span style="color: #7F0055; font-weight: bold;">extends</span> <span style="color: #000000;">Thread</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #000000;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		@Override</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #7F0055; font-weight: bold;">public</span> <span style="color: #7F0055; font-weight: bold;">void</span> run<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #000000;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">			<span style="color: #808080; font-style: italic;">// Tant que le thread n'est pas interrompu...</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">			<span style="color: #7F0055; font-weight: bold;">long</span> time = 0L;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">			<span style="color: #7F0055;font-weight: bold;">while</span> <span style="color: #000000;">&#40;</span><span style="color: #000000;">!</span> isInterrupted<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span><span style="color: #000000;">&#41;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">			<span style="color: #000000;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				time = <span style="color: #000000;">System</span>.<span style="color: #000000;">currentTimeMillis</span><span style="color: #000000;">&#40;</span><span style="color: #000000;
">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				<span style="color: #7F0055; font-weight: bold;">try</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				<span style="color: #000000;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">					<span style="color: #808080; font-style: italic;">// Une seconde d'attente</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">					<span style="color: #000000;">Thread</span>.<span style="color: #000000;">sleep</span><span style="color: #000000;">&#40;</span><span style="color: #cc66cc;">1000</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				<span style="color: #000000;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				<span style="color: #7F0055; font-weight: bold;">catch</span> <span style="color: #000000;">&#40;</span><span style="color: #000000;">InterruptedException</span> e<span style="color: #000000;">&#41;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				<span style="color: #000000;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">					<span style="color: #000000;">System</span>.<span style="color: #000000;">out</span>.<span style="color: #000000;">println</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;J'ai été interrompu !&quot;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">					<span style="color: #808080; font-style: italic;">// Activation du flag d'interruption</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">					<span style="color: #000000;">Thread</span>.<span style="color: #000000;">currentThread</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>.<span style="color: #000000;">interrupt</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				<span style="color: #000000;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">				<span style="color: #000000;">System</span>.<span style="color: #000000;">out</span>.<span style="color: #000000;">println</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;Boucle de &quot;</span> + <span style="color: #000000;">&#40;</span><span style="color: #000000;">System</span>.<span style="color: #000000;">currentTimeMillis</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span> - time<span style="color: #000000;">&#41;</span> + <span style="color: #888888;">&quot;ms&quot;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">			<span style="color: #000000;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		<span style="color: #000000;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #000000;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #000000;">&#125;</span></div></li></ol></pre> <p>Résultat&nbsp;:</p> <pre> Début de la boucle, attente prévue 1000ms Fin de la boucle, attente réelle 1000ms Début de la boucle, attente prévue 1000ms Fin de la boucle, attente réelle 1000ms Début de la boucle, attente prévue 1000ms Fin de la boucle, attente réelle 1000ms Début de la boucle, attente prévue 1000ms &lt;saisie dans la console&gt; Interruption ! J'ai été interrompu ! Fin de la boucle, attente réelle 479ms </pre>