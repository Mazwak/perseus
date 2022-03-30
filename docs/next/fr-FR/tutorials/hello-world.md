# Bonjour le monde

Commençons avec Perseus !

_Pour suivre ici, vous devez être familier avec Rust, sur lequel vous pouvez en savoir plus [ici](https://rust-lang.org). Vous devriez également installé `rust` et `cargo`._

Pour commencer, créez un nouveau dossier pour votre projet, appelons-le `my-perseus-app`. Maintenant, créez un fichier `Cargo.toml` dans ce dossier. Cela indique à Rust quels packages vous souhaitez utiliser dans votre projet et certaines autres métadonnées. Mettez ce qui suit à l’intérieur :

```toml
{{#include ../../../examples/comprehensive/tiny/Cargo.toml.example}}
```

<details>
<summary>Que font ces dépendances ?</summary>

- `perseus` -- le module de base pour Perseus
- [`sycamore`](https://github.com/sycamore-rs/sycamore) -- l’incroyable système sur lequel Perseus est bâti, cela vous permet d’écrire des applications web réactives en Rust

Notez que nous avons configuré ces dépendances pour qu’elles mettent automatiquement à jour les _versions correctives_, ce qui signifie que nous obtiendrons automatiquement des corrections de bogues, mais nous n’obtiendrons aucune mise à jour susceptible de casser notre application !

</details>

Maintenant, créez un fichier `index.html` à la racine de votre projet et placez-y ce qui suit :

```html
{{#include ../../../examples/comprehensive/tiny/index.html}}
```

<details>
<summary>N’ai-je pas besoin d’un fichier `index.html` ?</summary>

Avec les versions de Perseus antérieures à la v0.3.4, un fichier `index.html` était nécessaire pour que Perseus sache comment s’afficher dans les navigateurs de vos utilisateurs, cependant, cela n’est plus nécessaire, car Perseus a maintenant une _vue d’index_ intégrée par défaut, avec la possibilité de fournir la vôtre via le code `index.html` ou Sycamore !

Pour les exigences de toutes les vues d’index que vous créez, voir ci-dessous.

</details>

Maintenant, créez un nouveau répertoire appelé `src` et ajoutez un nouveau fichier à l’intérieur appelé `lib.rs`. Mettez ce qui suit à l’intérieur :

```rust
{{#include ../../../examples/comprehensive/tiny/src/lib.rs}}
```

<details>
<summary>Comment ça marche ?</summary>

Tout d’abord, nous importons certaines choses qui seront utiles :

- `perseus::{Html, PerseusApp, Template}` -- le trait `Html`, qui permet à votre code d’être générique afin qu’il puisse être utilisé soit sur le serveur soit dans le navigateur (vous le verrez tout au long du code Sycamore écrit pour Perseus); la `struct` `PerseusApp`, qui est la façon de représenter une application Perseus ; la `struct` `Template`, qui représente un _modèle_ dans Perseus (qui peut créer des pages, comme vous l’apprendrez bientôt -- c’est le bloc fondamental de Perseus)
- `sycamore::view` -- La macro `view!` de Sycamore, qui vous permet d’écrire du code simili-HTML en Rust

Perseus utilisait une macro appelée `define_app!` pour définir votre application, elle est devenue obsolète et a été remplacée par une `struct` constructeur plus moderne, qui dispose de méthodes que vous pouvez utiliser pour ajouter des fonctionnalités supplémentaires à votre application (comme l’internationalisation ). C’est `PerseusApp`, et ici, nous ajoutons juste un modèle avec l’appel `.template()` (que vous exécuterez chaque fois que vous voudrez ajouter un nouveau modèle à votre application). Ici, nous créons un modèle très simple appelé `index`, un nom de modèle spécial qui liera ce modèle à la racine de votre application, ce sera la page d’arrivée. Nous définissons ensuite le code de vue pour ce modèle avec la méthode `.template()` sur la `struct` `Template`, auquel nous fournissons une closure simple qui renvoie un `view!` Sycamore, qui génére juste un élément de paragraphe HTML (`<p>Hello World !</p>` avec le balisage HTML habituel). Habituellement, nous fournirrions ici une fonction étoffée qui pourrait faire beaucoup plus de choses (comme accéder aux états globaux), mais pour l’instant, nous allons garder les choses simples et agréables.

Dans la plupart des applications, les principales choses que vous définirez sur `PerseusApp` sont des `Template`s, cependant, lorsque vous passerez à la production, vous voudrez également définir des `ErrorPages`, qui indiquent à Perseus quoi faire si votre application atteint une page inexistante (une erreur 404 introuvable) ou similaire. Pour un développement rapide cependant, Perseus fournit une série de pages d’erreur prédéfinies (mais si vous essayez de les utiliser implicitement en production, vous obtiendrez un message d’erreur).

Notez également que nous définissons ce `PerseusApp` dans une fonction appelée `main`, mais vous pouvez l’appeler comme vous voulez, tant que vous mettez `#[perseus::main]` juste avant, ce qui la transforme en quelque chose que Perseus peut trouver (spécifiquement, une fonction spéciale nommée `__perseus_entrypoint`).

</details>

Installez maintenant la CLI Perseus avec `cargo install perseus-cli` (vous aurez besoin de `wasm-pack` pour permettre à Perseus de construire votre application, utilisez `cargo install wasm-pack` pour l’installer) pour vous faciliter la vie, et déployez votre application sur <http://localhost:8080> en exécutant `perseus serve` à la racine de votre projet ! Cela prendra un certain temps la première fois, car il doit récupérer toutes vos dépendances et compiler votre application.

<details>
<summary>Pourquoi ai-je besoin d’une CLI ?</summary>

Perseus est un système _très_ complexe, et, si vous deviez écrire toute cette complexité vous-même, cet exemple _Hello World!_ ressemblerait plus à 1200 lignes de code qu’à 12 ! La CLI vous permet de vous défaire de toute cette complexité dans un répertoire que vous avez peut-être remarqué appelé `.perseus/`. Si vous jetez un coup d’œil à l’intérieur, vous trouverez en fait trois crates (bibliothèques Rust) : une pour votre application, une autre pour le serveur qui met à disposition votre application et une dernière pour le constructeur qui compile votre application. Ce sont eux qui exécutent réellement votre application et ils importent le code que vous avez écrit. Ils s’interfacent avec la `PerseusApp` que vous définissez pour que tout cela fonctionne.

Lorsque vous exécutez `perseus serve`, le répertoire `.perseus/` est créé et ajouté à votre `.gitignore`, puis trois étapes se déroulent en parallèle (elles sont affichées dans votre terminal) :

- _🔨 Génération de votre application_ -- ici, votre application est transformée en une série de fichiers statiques dans `.perseus/dist/static`, ce qui rend votre application ultra-rapide (les pages de votre application sont prêtes avant même qu’elle ne soit déployée, ce qui s’appelle _génération de site statique_, ou SSG)
- _🏗️ Compiler votre application en Wasm_ -- ici, votre application est compilée en [WebAssembly](https://webassembly.org), ce qui permet à un langage de programmation de bas niveau comme Rust de s’exécuter dans le navigateur
- _📡 Compiler un serveur_ -- ici, Perseus compile le serveur interne basé sur votre code et se prépare à servir votre application (notez qu’une application aussi simple peut en fait utiliser [l’exportation statique](:reference/exporting), mais nous regarderons ça plus tard)

La première fois que vous exécutez cette commande, cela peut prendre un certain temps pour que tout soit prêt, mais après cela, ce sera très rapide. Et, si vous n’avez pas changé de code (_du tout_) depuis la dernière fois que vous l’avez exécuté, vous pouvez exécuter `perseus serve --no-build` pour exécuter le serveur pratiquement instantanément.

</details>

Une fois cela fait, ouvrez <http://localhost:8080> dans n’importe quel navigateur moderne (pas Internet Explorer...), et vous devriez voir _Hello World!_ affiché à l’écran ! Si vous essayez d’aller sur <http://localhost:8080/about> ou toute autre page, vous devriez voir un message vous indiquant que la page n’a pas été trouvée.

Toutes nos félicitations ! Vous venez de créer votre toute première application Perseus ! Vous pouvez voir le code source de cette section [ici](https://github.com/arctic-hen7/perseus/tree/main/examples/comprehensive/tiny).

## Aller plus loin

La section suivante crée une application légèrement plus réaliste avec plus d’un fichier, qui vous montrera comment une application Perseus est généralement structurée.

Après cela, vous apprendrez comment fonctionnent différentes fonctionnalités de Perseus, comme la _génération incrémentielle_ (qui vous permet de créer des pages à la demande lors de l’exécution) !

### Alternatives

Si vous en arrivez ici et que vous n’êtes pas si emballé par Perseus, voici quelques projets similaires en Rust :

- [Sycamore](https://github.com/sycamore-rs/sycamore) (sans Perseus) -- _Une bibliothèque réactive pour créer des applications Web en Rust et WebAssembly._
- [Yew](https://github.com/yewstack/yew) -- _Framework Rust/Wasm pour la création d’applications Web clientes._
- [Seed](https://github.com/seed-rs/seed) -- _Un framework Rust pour créer des applications Web._
- [Percy](https://github.com/chinedufn/percy) -- _Construisez des applications frontales de navigateur avec Rust + WebAssembly. Prend en charge le rendu côté serveur (SSR)._
- [MoonZoon](https://github.com/MoonZoon/MoonZoon) -- _Rust Fullstack Framework._
