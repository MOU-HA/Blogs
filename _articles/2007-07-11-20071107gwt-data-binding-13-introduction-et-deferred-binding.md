---
ID: 58
post_title: 'GWT Data Binding (1/3) : Introduction et Deferred Binding'
author: pqueinnec
post_date: 2007-07-11 12:09:00
post_excerpt: '<p>Ce tutorial a pour but d’expliquer le fonctionnement du Deferred Binding de GWT et de l’utilisation de la classe Generator pour créer des classes dynamiquement. L’exemple qui nous permettra d’illustrer ce tutorial est la création d’une classe utilitaire permettant d’accéder aux données contenues dans un objet à partir de leurs noms. Cette fonctionnalité est très pratique dans les problématiques de liaisons automatiques du modèle aux IHMs (data binding).</p>'
layout: post
permalink: http://blog.zenika-offres.com/?p=58
published: true
---
<p>Ce tutorial a pour but d’expliquer le fonctionnement du Deferred Binding de GWT et de l’utilisation de la classe Generator pour créer des classes dynamiquement. L’exemple qui nous permettra d’illustrer ce tutorial est la création d’une classe utilitaire permettant d’accéder aux données contenues dans un objet à partir de leurs noms. Cette fonctionnalité est très pratique dans les problématiques de liaisons automatiques du modèle aux IHMs (data binding).</p>
<!--more-->
<h3>Deferred Binding</h3> <p>En Java, la réflexion est un mécanisme très puissant. Supposons que l’on ait les classes <code>Employe</code> et <code>Visiteur</code> qui sont des sous-classes de la classe <code>Personne</code> et que l’on veuille qu’un objet personne désigne une instance d’<code>Employe</code> ou de <code>Visiteur</code> en fonction du contexte.</p> <p>En Java on utilisera la réflexion comme ceci&nbsp;:</p> <pre class="java code java" style="font-family:inherit"><ol><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #808080; font-style: italic;">// status contient “Employe” ou “Visiteur”</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #000000;">String</span> status = getStatus<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">Personne personne = <span style="color: #000000;">&#40;</span>Personne<span style="color: #000000;">&#41;</span> <span style="color: #7F0055; font-weight: bold;">Class</span>.<span style="color: #000000;">forName</span><span style="color: #000000;">&#40;</span>status<span style="color: #000000;">&#41;</span>.<span style="color: #000000;">newInstance</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</div></li></ol></pre> <p>Ainsi, l’instance désignée par l’objet personne est choisie dynamiquement en fonction de la chaîne de caractère status.</p> <p>Pour plusieurs raisons, la réflexion n’est pas possible avec GWT. La première est que dans un souci d’optimisation, seul les méthodes et classes utilisées dans l’application sont traduites en JavaScript lors de la compilation. L’autre raison est qu’il n’est pas possible de charger une classe dynamiquement en JavaScript.</p> <p>Pour compenser ce manque GWT possède le Deferred Binding qui consiste à effectuer l’insertion de la classe demandée à la compilation et non durant l’exécution du programme. L’appel d’une instance se fait grâce à la commande <code>GWT.create(Classe.class)</code>.</p> <p>Il n’est cependant pas possible d’appeler une classe en fonction d’une chaine de caractère (’Classe’) comme avec la réflexion. Le nom de la classe doit être décrit de manière litérale (<code>Class.class</code>). On peut alors se questionner sur la différence entre <code>GWT.create(Classe.class)</code> et <code>new Classe()</code>.</p> <p>La différence se situe au niveau de la compilation. L’appel <code>new Class()</code> est directement traduit en JavaScript, alors que l’appel <code>GWT.create()</code> passe à travers un Generator qui spécifie quelle classe doit être instanciée.</p>