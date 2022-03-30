# Votre deuxième application

Cette section couvrira la création d’une application plus réaliste que la section _Hello World!_, avec une structure appropriée et plusieurs modèles.

Si apprendre en lisant n’est pas vraiment votre truc, ou si vous souhaitez une référence, vous pouvez voir tout le code dans [ce dépot](https://github.com/arctic-hen7/perseus/tree/main/examples/core/basique) !

## Préparatifs

Tout comme l’application _Hello World!_, nous allons commencer par créer un nouveau répertoire pour le projet, peut-être `my-second-perseus-app` (ou vous pouvez faire preuve d’imagination…). Ensuite, nous allons créer un nouveau fichier `Cargo.toml` et le remplir avec ce qui suit :

```toml
{{#include ../../../examples/core/basic/Cargo.toml.example}}
```

La seule différence entre ceci et le précédent `Cargo.toml` que nous avons créé sont deux nouvelles dépendances :

- [`serde`](https://serde.rs) -- une bibliothèque Rust très utile pour sérialiser/désérialiser les données
- [`serde_json`](https://github.com/serde-rs/json) -- L’intégration de Serde pour JSON, qui nous permet de transmettre des propriétés pour des pages plus avancées dans Perseus (vous pouvez ne pas l’utiliser explicitement, mais vous en aurez besoin comme dépendance de certaines macros Perseus)

## `lib.rs`

Comme dans toutes les applications Perseus, `lib.rs` est la façon dont nous communiquons avec la CLI et lui expliquons comment fonctionne notre application. Mettez le contenu suivant dans `src/lib.rs` :

```rust
{{#include ../../../examples/core/basic/src/lib.rs}}
```

Ce code est assez différent de votre première application, alors voyons comment cela fonctionne.

Tout d’abord, nous définissons deux autres modules dans notre code : `error_pages` (dans `src/error_pages.rs`) et `templates` (dans `src/templates`). Ne vous inquiétez pas, nous allons les créer dans un instant. Le reste du code crée une nouvelle application avec deux modèles, tous deux issus du module "templates". Plus précisément, nous fournissons la fonction `.template()` avec une autre fonction qui produit notre modèle, ce qui nous permet de conserver le code de chaque modèle dans un fichier séparé.

Nous utilisons également `.error_pages()` ici pour indiquer à Perseus comment gérer les erreurs dans notre application (comme une page inexistante), et nous les placerons dans le module `error_pages`. Notez que vous n’avez pas à le faire en développement, Perseus a un ensemble de valeurs par défaut qu’il peut utiliser, mais vous ne pouvez pas les utiliser en production, vous devrez donc créer des pages d’erreur à un moment donné.

## La gestion des erreurs

Avant d’aborder la partie intéressante de la création des pages utiles de l’application, nous devons configurer des pages d’erreur, ce que nous ferons dans `src/error_pages.rs` :

```rust
{{#include ../../../examples/core/basic/src/error_pages.rs}}
```

La première chose à noter ici est l’importation de [`Html`](https://docs.rs/sycamore/0.7/sycamore/generic_node/trait.Html.html), que nous définissons comme paramètre de type sur les fonctions `get_error_pages`. Cela garantit que nous pouvons compiler ces vues sur le client ou le serveur tant qu’elles ciblent le HTML (Sycamore peut également cibler d’autres modèles de formattage pour des systèmes complètement différents, comme les applications de bureau MacOS).

Dans cette fonction, nous définissons également une page d’erreur différente pour une erreur 404, qui se produit lorsqu’un utilisateur tente d’accéder à une page qui n’existe pas. La page de secours (avec laquelle nous initialisons `ErrorPages`) est la même que la dernière fois et sera appelée pour toute erreur autre qu’un _404 Not Found_. Notez que les pages d’erreur que nous définissons ici sont extrêmement similaires aux pages par défaut de Perseus, et, dans une vraie application, vous créeriez probablement quelque chose de beaucoup plus plaisant !

## `index.rs`

Il est temps de créer la première page de cette application ! Mais d’abord, nous devons nous assurer que l’importation dans `src/lib.rs` de `mod templates;` fonctionne, ce qui nous oblige à créer un nouveau fichier `src/templates/mod.rs`, qui déclare `src/templates` en tant que module dans votre crate avec son propre code (c’est ainsi que les dossiers fonctionnent dans les projets Rust). Ajoutez ce qui suit à ce fichier :

```rust
{{#include ../../../examples/core/basic/src/templates/mod.rs}}
```

Il est courant d’avoir un fichier (ou même un dossier) pour chaque _template_, qui est légèrement différent d’une page (expliqué plus en détail plus tard), et cette application a deux pages : une page d’arrivée (index) et une page « à propos ».

Commençons par la page d’arrivée. Créez un nouveau fichier `src/templates/index.rs` et mettez ce qui suit à l’intérieur :

```rust
{{#include ../../../examples/core/basic/src/templates/index.rs}}
```

Ce code est _beaucoup_ plus complexe que l’exemple _Hello World!_, alors examinons-le attentivement.

Tout d’abord, nous importons une tonne de choses :

- `perseus`
  - `RenderFnResultWithCause` -- voir ci-dessous pour une explication
  - `Modèle` -- comme avant
  - `Html` -- comme avant (cela provient de Sycamore, mais est réexporté par Perseus pour plus de commodité)
  - `http::header::{HeaderMap, HeaderName}` -- quelques types pour ajouter des en-têtes HTTP à notre page
  - `SsrNode` -- La représentation par Sycamore d’un nœud qui ne sera rendu que sur le serveur (cela vient de la crate `sycamore`, mais Perseus le réexporte pour plus de commodité)
- `serde`
  - `Serialize` -- un trait pour les `struct`s qui peuvent être transformées en chaîne de caractères (comme JSON)
  - `Deserialize` -- un trait pour les `struct`s qui peuvent être _dé_sérialisées à partir d’une chaîne de caractères (comme JSON)
- `sycomore`
  - `component` -- une macro qui transforme une fonction en un composant Sycamore
  - `view` -- la macro `view!`, comme avant
  - `View` -- le résultat de la macro `view!`

Ensuite, nous définissons un certain nombre de fonctions différentes et une `struct`, les deux étant expliqués dans les sections ci-dessous.

### `IndexPageState`

Cette `struct` représente l’_état_ de la page d’index. Comme mentionné dans l’explication des [principes fondamentaux](:core-principles) de Perseus, c’est fondamentalement un système permettant de transformer un état et une vue en une interface utilisateur. Dans ce cas, notre page d’index affichera un message d’accueil généré au moment de la compilation. Cela signifie que notre code de la vue sera générique par rapport au message d’accueil que nous générons (que vous pouvez voir dans notre code `view! {...}` dans `index_page()`). Perseus nous permet de générer un état de [plusieurs manières](:reference/strategies), et dans ce cas, nous utiliserons la plus simple : [build state](:reference/strategies/build-state), qui exécute simplement une fonction personnalisée lorsque nous compilons notre application qui génère une instance de notre état, qui est représenté par cette `struct`.

Notez que nous utilisons également la macro `#[perseus::make_rx(IndexPageStateRx)]` ici pour rendre notre état _réactif_. Ce que fait cette macro est de prendre notre `struct` et d’en produire une nouvelle version appelée `IndexPageStateRx` qui a exactement les mêmes champs, mais chacun enveloppé dans un `Signal` Sycamore, ce qui rend chaque champ réactif. Cela signifie que notre page peut modifier son état à un endroit et tous les autres endroits qui utilisent cet état seront automatiquement mis à jour via le système de réactivité de Sycamore ! Vous pouvez en savoir plus sur les idées derrière cela [ici](:reference/state/rx).

### `index_page()`

Il s’agit du véritable composant qui générera une interface utilisateur pour votre page. Perseus vous permet de fournir une _fonction de modèle_ comme celle-ci en tant que simple fonction Rust qui prend en compte l’état de votre page et produit un `View<G>` Sycamore (encore une fois, `G` est ambiant ici à cause de la macro procédurale). Cependant, il y a _beaucoup_ de travail en coulisses pour rendre votre état réactif, l’enregistrer avec Perseus, gérer [l’état global](:reference/state/global) et configurer un composant Sycamore utilisable par le reste du code Perseus. Tout cela est fait avec l’une des deux macros d’attribut : `#[perseus::template(...)]` ou `#[perseus::template_rx]`. Dans les versions précédentes de Perseus, vous utilisiez la première, ce qui vous donnait une instance non réactive de votre état (dans notre cas, `IndexPageState`). Cependant, depuis la version 0.3.3, il est recommandé d’utiliser la seconde, qui vous donne une version réactive (dans notre cas, `IndexPageStateRx`) et peut gérer des fonctionnalités plus avancées de la [plateforme d’état réactif](:reference/state/rx) de Perseus, que nous utilisons donc ici. De plus, `template_rx` gère des choses comme les composants Sycamore en interne pour vous, minimisant la quantité de code que vous devez réellement écrire.

Notez que `index_page()` prend `IndexPageStateRx` comme argument, auquel il peut ensuite accéder dans la `view!`. Il s’agit du système d’interpolation de Sycamore, que vous pouvez lire [ici](<https://sycamore-rs.netlify.app/docs/basics/template>), mais tout ce que vous devez savoir, c’est que c’est essentiellement transparent et fonctionne exactement comme vous pouvez vous y attendre (rappelez-vous cependant que, parce que nous utilisons la macro `template_rx`, nous devons utiliser un `struct` d’état réactif, que nous générons avec la macro `make_rx`, afin que tous nos champs soient enveloppés dans des `Signal` s, et c’est pourquoi nous utilisons `.get()`).

La seule autre chose que nous faisons ici est de définir un `<a>` (un lien HTML) vers `/about`. Ce lien, et tous les autres que vous définissez, seront automatiquement détectés par les systèmes de Sycamore, qui les transmettront à la logique de routage de Perseus, ce qui signifie que vos utilisateurs **ne quittent jamais la page**. De cette façon, Perseus ne récupère que le contenu qui doit changer et donne à vos utilisateurs le sentiment d’une application ultra-rapide et légère.

_Remarque : les liens externes en seront automatiquement exclus, et vous pouvez en exclure manuellement en ajoutant `rel="external"` si besoin._

### `head()`

Cette fonction est très similaire à `index_page()`, sauf qu’il ne s’agit pas d’un composant Sycamore à part entière, elle renvoie simplement un `view! {}` à la place, et cela n’utilise qu’une version non réactive de votre état. Cela sert à définir le contenu de `<head>`, qui sont des métadonnées pour votre site Web, comme son `<title>`. Comme vous pouvez le voir, ceci récupère les mêmes propriétés que `index_page()`, mais nous ne les utilisons pas dans cet exemple. La macro `#[perseus::head]` dit à Perseus de faire un travail dans les coulisses qui est très similaire à celui fait avec `index_page`, mais spécialisé pour `<head>`.

Ce qu’il est vraiment important de noter à propos de cette fonction, c’est qu’elle n’est générée que dans un `SsrNode`, ce qui signifie que vous ne pouvez pas utiliser la réactivité ici ! Tout ce qui est généré la première fois sera transformé en `String` puis interpolé statiquement dans le `<head>` du document. Cela signifie également que cela ne s’exécute que sur le serveur (si vous souhaitez le modifier sur le client, vous devrez le faire manuellement).

La différence entre les métadonnées définies ici et les métadonnées définies dans votre fichier `index.html`, ces que les secondes s’appliqueront à chaque page, alors que les premières ne s’appliqueront qu’au modèle. Du coup, c’est plus utile pour des choses comme les titres, alors que vous utiliserez `index.html` pour importer des feuilles de style ou des outils d’analyse.

Si vous inspectez le code source du HTML dans votre navigateur, vous trouverez un gros commentaire dans `<head>` qui dit `<!--PERSEUS_INTERPOLATED_HEAD_BEGINS-->`, qui sépare les choses qui devraient rester les mêmes sur toutes les pages des éléments qui doivent être mis à jour à chaque page.

### `get_template()`

Cette fonction est celle que nous appelons dans `lib.rs`, et elle combine tout le reste de ce fichier pour produire un véritable `Template` Perseus à utiliser. Notez le nom du modèle, `index`, que Perseus interprète comme spécial, ce qui entraîne la génération de ce modèle à `/` (la page d’arrivée).

Le système de modèles de Perseus est extrêmement polyvalent, et ici nous l’utilisons pour définir notre page elle-même via `.template()`, et pour définir une fonction qui modifiera le document `<head>` (ce qui nous permet d’ajouter un titre) avec `.head()`. Notamment, nous utilisons également la stratégie de génération _build state_, qui indique à Perseus d’appeler la fonction `get_build_state()` lorsque votre application se compile pour obtenir un état (une instance de `IndexPageState` pour être précis). Plus sur cela dans un instant.

#### `.template()`

Cette fonction est celle que Perseus appellera lorsqu’il voudra générer votre modèle (ce qu’il fait plus souvent que vous ne le pensez). Si vous avez utilisé la macro `#[perseus::template_rx]` ou `#[perseus::template(...)]` sur `index_page()`, vous pouvez fournir `index_page` directement ici, mais cela peut être utile de comprendre ce que fait cette macro.

Dans les coulisses, cette macro transforme votre fonction `index_page()` pour prendre les propriétés en tant que `Option<String>` au lieu de `IndexPageState`, car Perseus transmet en fait vos propriétés en interne en tant que `String`s. Au début, cela peut sembler bizarre, mais cela évite quelques problèmes courants qui augmenteraient la taille finale de l’exécutable de votre Wasm et rendraient le chargement de votre site Web très long. Fait intéressant, il est également plus performant d’utiliser des `String`s partout, car nous devons de toute façon effectuer cette conversion lorsque nous envoyons vos propriétés au navigateur d’un utilisateur. De plus, la macro `#[perseus::template_rx]` gérera la configuration et l’interaction avec les fonctionnalités les plus complexes de la [plate-forme d’état réactif](:reference/state/rx) de Perseus.

Si tout cela vous laisse perplexe, ne vous inquiétez pas, c’est ce que fait Perseus en arrière plan, et que vous aviez l’habitude de faire à la main ! Les macros `#[perseus::template(...)]`/`#[perseus::template_rx]` font tout cela pour vous.

#### `.head()`

C’est juste l’équivalent de `.template()` pour la fonction `head()`, et cela fait exactement la même chose. La seule chose à noter ici est que les propriétés attendues sont à nouveau en tant que `Option<String>`, et celles-ci sont automatiquement désérialisées par la macro `#[perseus::head]` que nous avons utilisée sur `head()` plus tôt.

### `get_build_props()`

Cette fonction fait partie des ingrédients secrets de Perseus (en fait _open_ sauce NdT : jeux de mots sur `secret sauce` en anglais), et elle sera appelée lorsque la CLI compile votre application pour créer les propriétés que le modèle utilise (il attend une chaîne de caractères, d’où la sérialisation). Ici, nous avons simplement codé en dur une salutation à utiliser, mais la vraie puissance vient lorsque vous commencez à utiliser le fait que cette fonction est _asynchrone (async)_. Vous pouvez interroger une base de données pour obtenir une liste d’articles de blog, ou extraire une page de documentation Markdown et l’analyser, les possibilités sont infinies !

Cette fonction renvoie un type plutôt spécial, `RenderFnResultWithCause<IndexPageState>`, qui déclare que votre fonction renverra `IndexPageState` si elle réussit, et une erreur spéciale si elle échoue. Cette erreur peut être n’importe quoi (c’est une `Box<dyn std::error::Error + Send + Sync>` en interne), mais elle aura également un blâme qui lui sera attribué qui permet de savoir si c’était le serveur ou le client qui a causé l’erreur, ce qui aura un impact sur le code d’état HTTP final (par exemple 404, 500, etc.). Vous pouvez utiliser la macro `blame_err!` pour créer facilement ces erreurs, mais chaque fois que vous utiliserez `?` dans les fonctions qui renvoient ce type, vous utiliserez simplement la valeur par défaut consistant à blâmer le serveur et à renvoyer le code d’état HTTP _500 Internal Server Error_.

Il peut sembler un peu inutile de blâmer le client dans le processus de construction, mais la raison pour laquelle cela peut arriver est que, dans des utilisations plus avancées de Perseus (en particulier la [génération incrémentielle](:reference/strategies/incremental)), cette fonction pourrait être appelée à la suite de la demande d’un client avec des paramètres qu’il fournit et qui peuvent être invalides. Fondamentalement, sachez que c’est une chose qui est importante dans les cas d’utilisation les plus complexes de Perseus.

Ce `#[perseus::autoserde(build_state)]` est également quelque chose que vous verrez assez souvent (mais pas dans les anciennes versions de Perseus). C’est une macro pratique qui sérialise automatiquement le retour de votre fonction dans une `String` pour que Perseus l’utilise en interne, ce qui n’est simplement que l’opposé de l’annotation que nous avons utilisée précédemment sur `index_page()`. Techniquement, vous n’en avez pas besoin, mais cela élimine du code _boilerplate_ que vous n’avez pas besoin d’écrire vous-même.

## `about.rs`

Bien ! Nous avons passé un cap, et maintenant il est temps de définir la page (beaucoup plus simple) `/about`. Créez `src/templates/about.rs` et mettez ce qui suit à l’intérieur :

```rust
{{#include ../../../examples/core/basic/src/templates/about.rs}}
```

C’est exactement la même chose que `index.rs`, sauf que nous n’avons aucun état à gérer et que nous n’avons pas besoin de générer quoi que ce soit de spécial au moment de la compilation (mais Perseus génèrera quand même cette page en HTML statique à la compilation, prête à être envoyée à vos utilisateurs).

## L’utiliser

`perseus export -sw`

C’est tout. Chaque fois que vous créez une application Perseus, c’est tout ce que vous avez à faire.

*Remarque : Étant donné que cette application est très simple et n’utilise aucune fonctionnalité nécessitant un serveur, nous pouvons utiliser [l’exportation statique](:reference/exporting). Pour certaines applications plus complexes, vous devrez utiliser `perseus serve -w` pour lancer un serveur complet.*

Une fois cette opération terminée, votre application sera accessible à l’adresse <http://localhost:8080> ! Notez que si vous le désirez, vous pouvez changer l’hôte/port avec les variables d’environnement `PERSEUS_HOST`/`PERSEUS_PORT` (par exemple, vous définiriez l’hôte sur `0.0.0.0` si vous voulez que d’autres personnes sur votre réseau puissent accéder à votre site).

Rendez-vous sur <http://localhost:8080> dans n’importe quel navigateur moderne et vous devriez voir votre message d’accueil "Hello World !" au-dessus d’un lien vers la page « À propos » ! Si vous cliquez sur ce lien, vous serez redirigé vers une page indiquant simplement "À propos". Mais remarquez que votre navigateur ne semble jamais accéder à une nouvelle page (l’onglet n’affiche pas d’icône de chargement) ? C’est le _app shell_ de Perseus en action, qui intercepte la navigation vers d’autres pages et la fait se dérouler de manière transparente, ne récupérant que le strict minimum pour que la nouvelle page se charge. Le même comportement se produira si vous utilisez les boutons avant/arrière de votre navigateur.

Vous pouvez également essayer de modifier une partie du code de votre application (comme le message d’accueil généré), et vous verrez que votre application se reconstruira automatiquement. Lorsque c’est fait, votre navigateur rechargera la nouvelle version de votre application (en gardant autant de l’ancien état que possible, ce qui signifie que vous pouvez continuer à travailler sans perdre votre TODO, voir [ici](:reference/state/hsr) pour les détails ) ! Sous le capot, ce processus est similaire au _rechargement de module à chaud_ que la plupart des frameworks JavaScript effectuent, mais il est en fait encore plus avancé et résilient.

<details>
<summary>Pourquoi un "navigateur moderne" ?</summary>

### Compatibilité du navigateur

Perseus est compatible avec tous les navigateurs prenant en charge Wasm, qui sont les navigateurs les plus modernes comme Firefox et Chrome. Cependant, les navigateurs obsolètes comme Internet Explorer ne fonctionneront avec aucune application Perseus, sauf si vous utilisez _polyfill_ pour prendre en charge WebAssembly.

_Remarque : Techniquement, il est possible de « compiler » Wasm en JavaScript, et nous envisageons de le prendre en charge dans Perseus pour les sites qui doivent cibler de très anciens navigateurs. Pour le moment cependant, cela n’est pas pris en charge par Perseus._

</details>

## Aller plus loin

Toutes nos félicitations! Vous êtes maintenant sur la bonne voie pour créer des applications Web hautement performantes en Rust ! Les sections suivantes de ce livre ont un style plus référenciel et ne vous guideront pas dans la création d’une application, mais elles se concentreront plutôt sur des fonctionnalités spécifiques de Perseus qui peuvent être utilisées pour créer des systèmes extrêmement puissants.

Alors allez-y, et construisez !
