# Qu’est-ce que Perseus ?

Si vous connaissez [NextJS](https://nextjs.org), Perseus est la même chose pour Wasm. Si vous connaissez [SvelteKit](https://kit.svelte.dev), c’est la même chose pour [Sycamore](https://github.com/sycamore-rs/sycamore).

Si rien de tout cela n’a de sens, cette section est faite pour vous ! Si vous n’êtes pas d’humeur pour une longue lecture, il y a un résumé en bas de cette page !

### Développement Web Rust

[Rust](https://www.rust-lang.org/) est un langage de programmation extrêmement puissant, mais je vais en laisser l’introduction [à ses développeurs](<https://www.rust-lang.org> /).

[WebAssembly](https://webassembly.org) (en abrégé Wasm) est comme un langage de programmation de bas niveau pour votre navigateur. C’est révolutionnaire, car cela permet de créer des sites Web et des applications Web dans des langages de programmation autres que JavaScript. En outre, c’est [vraiment rapide](https://medium.com/@torch2424/webassembly-is-fast-a-real-world-benchmark-of-webassembly-vs-es6-d85a23f8e193) (généralement plus de 30 % plus rapide que JS).

Mais développer directement pour le Web avec Rust en utilisant quelque chose comme [`web-sys`](https://docs.rs/web-sys) n’est pas une expérience extraordinaire, il est généralement admis dans la communauté de développement Web que l’expérience des développeurs et la productivité sont grandement améliorées en ayant un framework _réactif_. Abordons d’abord cela d’un point de vue JavaScript et HTML traditionnel.

Imaginez que vous vouliez créer un simple compteur. Voici comment vous pourriez le faire dans un framework non réactif (encore une fois, JS et HTML ici, pas encore de Rust):

```html
<p id="counter">0</p>
<br />
<button
    onclick="document.getElementById('counter').innerHTML = parseInt(document.getElementById('counter').innerHTML) + 1"
>
    Incrémenter
</button>
```

Si vous n’êtes pas familier avec HTML et JS, ne vous inquiétez pas. Tout ce que cela fait est de créer un paragraphe avec un nombre à l’intérieur, et l’incrémenter. Mais le problème est clair en termes d’expressivité : Pourquoi ne pouvons-nous pas simplement mettre une variable dans le paragraphe et la ré-évaluer lorsque nous incrémentons cette variable ? Eh bien, c’est ça la réactivité !

En JS, il existe des frameworks comme [Svelte](https://svelte.dev) et [ReactJS](https://reactjs.org) qui résolvent ce problème, mais ils sont tous limités de manière significative par le langage lui-même. JavaScript est lent, typé dynamiquement et [un chouïa bordélique](https://medium.com/netscape/javascript-is-kinda-shit-im-sorry-2e973e36fec4). Comme tout ce qui concerne le Web, changer les choses est vraiment difficile car les gens ont déjà commencé à les utiliser, et il y aura toujours _quelqu’un_ qui utilisera encore Internet Explorer, qui ne supporte pratiquement aucune norme Web moderne.

[Wasm](https://webassembly.org) résout tous ces problèmes en créant un format unifié dans lequel d’autres langages de programmation, comme Rust, peuvent être compilés pour l’environnement du navigateur. Cela rend les sites Web plus sûrs, plus rapides et le développement plus productif. L’équivalent de ces frameworks réactifs pour Rust en particulier serait des projets comme [Sycamore](https://sycamore-rs.netlify.app), [Seed](https://seed-rs.org) et [Yew] (<https://yew.rs>). Sycamore est la plus extensible et bas niveau de ces options, et elle est plus performante car elle n’utilise pas de [DOM virtuel](https://svelte.dev/blog/virtual-dom-is-pure-overhead) ( lien à propos de JS plutôt que Rust), et il a donc été choisi pour être la colonne vertébrale de Perseus. Voici à quoi pourrait ressembler ce compteur dans [Sycamore](https://sycamore-rs.netlify.app) (l’incrémentation a été déplacée dans une nouvelle closure pour plus de commodité) :

```rust
use sycamore::prelude::*;

let counter = Signal::new(0);
let increment = cloned!((counter) => move |_| counter.set(*counter.get() + 1));

view! {
    p { (counter.get()) }
    button(on:click = increment) { "Incrementer" }
}
```

Vous pouvez en savoir plus sur les incroyables possibilités de Sycamore [ici](https://sycamore-rs.netlify.app).

### Ça semble bon…

Mais il y a un hic à tout cela : la génération. Avec toutes ces approches dans Rust jusqu’à présent (à l’exception de quelques-unes mentionnées plus loin), toutes vos pages sont générées _dans le navigateur de l’utilisateur_. Cela signifie que vos utilisateurs doivent télécharger votre code Wasm et l’exécuter avant de voir quoi que ce soit sur leurs écrans. Non seulement cela augmente votre temps de chargement ([ce qui peut faire fuir les utilisateurs](https://medium.com/@vikigreen/impact-of-slow-page-load-time-on-website-performance-40d5c9ce568a)), cela réduit également votre classement dans les moteurs de recherche.

Cela peut être résolu grâce à la  _génération côté serveur_ (SSR, Server Side Rendering), ce qui signifie que nous générons les pages sur le serveur et les envoyons au client, ce qui signifie que vos utilisateurs voient quelque chose très rapidement, puis cela devient _interactif_ (utilisable) un instant plus tard. C’est mieux pour la rétention des utilisateurs (temps de chargement plus courts) et le référencement (SEO, Search Engine Optimization).

L’approche traditionnelle du SSR consiste à attendre une requête pour une page particulière (disons `/about`), puis à la générer sur le serveur et à l’envoyer au client. C’est ce que fait [Seed](https://seed-rs.org) (une alternative à Perseus). Cependant, cela signifie que le _temps pour le premier octet_ (TTFB, Time To First Byte) de votre site Web est plus lent, car l’utilisateur n’obtiendra _rien_ du serveur avant qu’il ait terminé la génération. En période de forte charge, cela peut augmenter les temps de chargement de manière inquiétante.

La solution à cela est la _génération de site statique_ (SSG, Static Site Generation), grâce à laquelle vos pages sont générées _au moment de la compilation_, et elles peuvent être envoyées presque instantanément pour toutes les demandes. Cette approche est fantastique et jusqu’à présent largement non implémentée en Rust. L’inconvénient est que vous n’obtenez pas autant de flexibilité, car vous devez tout rendre au moment de la compilation. Cela signifie que vous n’avez accès à aucune information d’identification de l’utilisateur ou à quoi que ce soit d’autre. Chaque page que vous générez statiquement doit être la même pour chaque utilisateur.

Perseus fournit SSR _et_ SSG prêt à l’emploi, ainsi que la possibilité d’utiliser les deux sur la même page, de regénérer des pages après un certain laps de temps (par exemple, pour mettre à jour une liste d’articles de blog toutes les 24 heures) ou en fonction de certaines conditions (par exemple si la somme de contrôle d’un fichier a changé), ou même pour générer statiquement des pages à la demande (la première demande est SSR, tout le reste est SSG), ce qui signifie que vous pouvez obtenir le meilleur de chaque monde et des temps de compilation plus rapides.

À notre connaissance, le seul autre framework au monde qui prend actuellement en charge cet ensemble de fonctionnalités est [NextJS](https://nextjs.org) (avec une concurrence croissante de [GatsbyJS](https://www.gatsbyjs.com) ), qui ne fonctionne qu’avec JavaScript. Perseus va au-delà pour Wasm en prenant en charge de toutes nouvelles combinaisons d’options de génération qui n’étaient pas disponibles auparavant, vous permettant de créer des sites Web et des applications Web optimisés de manière extrêmement efficace.

## Quelle vitesse en attendre ?

[Les benchmarks montrent](https://rawgit.com/krausest/js-framework-benchmark/master/webdriver-ts-results/table.html) que [Sycamore](https://sycamore-rs.netlify.app) est légèrement plus rapide que [Svelte](https://svelte.dev) par endroits, un des frameworks JS le plus rapide jamais créé. Perseus utilise Sycamore et [Actix Web](https://actix.rs) ou [Warp](https://github.com/seanmonstar/warp) (l’un ou l’autre est pris en charge), les serveurs Web parmi les plus rapides au monde. Fondamentalement, Perseus est construit sur les technologies les plus rapides et est lui-même conçu pour être rapide.

La vitesse des frameworks Web est souvent mesurée par les scores [Lighthouse](https://developers.google.com/web/tools/lighthouse), qui sont des notes sur 100 (plus c’est élevé, mieux c’est) qui mesurent une foule de choses, comme _temps de blocage total_, _premier affichage de contenu_ et _temps jusqu’à l’interactivité_. Tout est ensuite agrégé en une note finale et regroupé en trois tranches : 0-49 (lent), 50-89 (moyen) et 90-100 (rapide). Ce site Web, qui est construit avec Perseus, en utilisant [l’exportation statique](:exporting) et les [optimisations de taille](:deploying/size), obtient systématiquement un 100 sur le bureau et plus de 90 pour le mobile. Vous pouvez le voir par vous-même [ici](<https://developers.google.com/speed/pagespeed/insights/?url=https%3A%2F%2Farctic-hen7.github.io%2Fperseus%2Fen-US%2F&tab> =bureau) sur l’outil _PageSpeed ​​Insights_ de Google.

<details>
<summary>Pourquoi pas 100 sur mobile ?</summary>

La seule mesure qui empêche Perseus d’obtenir un score parfait constant sur mobile est le _temps de blocage total_, qui mesure le temps entre le moment où le premier contenu apparaît sur la page et le moment où ce contenu est interactif. Bien sûr, le code WebAssembly est utilisé pour cette partie (compilé à partir de Rust), et il n’est pas encore complètement optimisé sur de nombreux appareils mobiles. À mesure que les navigateurs mobiles s’améliorent dans l’analyse de WebAssembly, le TBT diminuera probablement davantage, passant de la plage moyenne à la plage verte (ce que nous voyons pour les systèmes de bureau, plus puissants).

</details>

Si vous souhaitez voir comment Perseus se comporte en termes de vitesse et d’un certain nombre d’autres fonctionnalités par rapport à d’autres frameworks, consultez la [page de comparaisons](comparaisons).

## Est-ce pratique ?

Perseus vise à être plus pratique que tout autre framework Web Rust en adoptant une approche similaire à celle de [ReactJS] (<https://reactjs.org>). Perseus lui-même est un système extrêmement complexe composé de nombreuses parties changeantes qui peuvent toutes être réunies pour créer quelque chose d’incroyable, mais la grande majorité des applications n’ont pas besoin de toute cette personnalisation, nous avons donc construit une interface en ligne de commande (CLI) qui gère toute cette complexité pour vous, vous permettant de vous concentrer entièrement sur le code de votre application.

En gros, voici votre flux de travail :

1. Créez un nouveau projet.
2. Définissez votre application en environ 12 lignes de code et quelques lignes de configuration.
3. Codez votre incroyable application.
4. Lancez `perseus serve`.

## À quel point est-ce stable ?

Perseus est considéré comme raisonnablement stable à ce stade, bien qu’il ne puisse pas encore être recommandé pour une utilisation en _production_. Cela dit, ce site Web est entièrement construit avec Perseus et Sycamore, et il fonctionne plutôt bien !

Pour l’instant cependant, Perseus est parfait pour tout ce qui n’est pas directement exposé à Internet, comme les outils internes, les projets personnels, etc. Ne l’utilisez pas pour faire fonctionner une centrale nucléaire, d’accord ?

## Résumé

Si tout cela était trop long, voici un bref résumé de ce que fait Perseus et pourquoi il est utile !

- JS est lent et un chouïa bordélique, [Wasm](https://webassembly.org) vous permet d’utiliser la plupart des langages de programmation, comme Rust, dans le navigateur, et est vraiment rapide
- Faire du développement web sans la réactivité est vraiment ennuyeux, donc [Sycamore](https://sycamore-rs.netlify.app) est super
- Perseus vous permet de rendre votre application sur le serveur, ce qui rend l’expérience client _vraiment_ rapide, et ajoute une tonne de fonctionnalités pour rendre cela possible, pratique et productif (même pour les applications vraiment compliquées)
