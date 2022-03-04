# Bundlers

src: <https://medium.com/@gimenete/how-javascript-bundlers-work-1fc0d0caf2da>
Bundlers zijn op hoog niveau tools die je javascript-bestanden zoals de naam implied samenbundeld in 1 bestand. Dit ?was/is? (volgens [caniuse](https://caniuse.com/es6-module-dynamic-import) hebben alle major browser wel al even support voor dynamic imports) nodig omdat zeker in het verleden het niet mogelijk was om JS modules te importeren of exporteren binnen een browser. Tegenwoordig is dat [niet meer het geval](https://stackoverflow.com/questions/67245509/why-do-we-still-need-module-bundlers-when-we-have-native-esm-support-in-browsers), maar het is nog steeds inefficient om veel individuele modules te downloaden met HTTP/1.1. Tegenwoordig [gebruikt bijna 50% van alle website al http/2](https://w3techs.com/technologies/details/ce-http2). Codifly lijkt nog HTTP/1.1 te gebruiken (waarom? vragen).

## Webpack

Webpack is een beetje de benchmark in dit onderzoek. Het is de meest mature en gebruikte bundler in de lijst met ongeveer 3x zoveel downloads dan de next most popular keuze, Rollup (+-20mil vs. +-6.5mil). Webpack heeft enige configuratie nodig om te werken (niet meer in versie 5).

**De core concepten van webpack zijn als volgt ([source](https://webpack.js.org/concepts/)):**

### Entry

Het ingangspunt voor webpack, waar hij zal beginnen met het uitbouwen van zijn interne dependency graph. Kortom gaat hij dus vanuit de entry point(s) uitzoeken van welke modules en libraries de applicatie afhankelijk is.

Dit kan dus een single entry point zijn:

```js
module.exports = {
  entry: './path/to/my/entry/file.js',
};
```

In dezelfde stijl als single entry, kan er ook met meerdere entry points gewerkt worden a.d.h.v. een array. Dit heeft wel als drawback dat ze in één chunk worden gecombineerd. Dit is dus niet flexibel of scalable:

```js
module.exports = {
  entry: ['./src/file_1.js', './src/file_2.js'],
  output: {
    filename: 'bundle.js',
  },
};
```

Ten slotte kan ook object syntax gebruikt worden. In object syntax kunnen onafhankelijk, individueel benoemde entry points gemaakt worden die in verschillende bundles worden gecompileerd. [Dit kan gecombineerd worden met de array syntax hierboven](https://stackoverflow.com/a/52170619/10780174):

```js
module.exports = {
  entry: {
    app: './src/app.js',
    adminApp: './src/adminApp.js',
  },
};
```

### Output

Deze is redelijk straightforward. In de output parameter declareren we hoe de bundel(s) genoemd zal/zullen worden, en waar ze zal/zullen geplaats worden:

```js
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js',
  },
};
```

### Loaders

Hier begint het interessant te worden. Webpack begrijpt out of the box alleen maar JavaScript en JSON. Aan de hand van de loaders kunnen we dan dealen met het 2de grote stuk van deze paper, transpilers. In de loaders configuratie kunnen we regels opmaken, zodat files met een bepaalde extensie worden getransformeed aan de hand een loader voor die extensie. Gewoon omdat er in deze paper vooral zal gekeken worden naar transpilers, betekent niet dat loaders niet veel meer kunnen doen. Om de dependencies van onze css te resolven gebruiken we `css-loader` (interpreteerd de css imports), dan kunnen we ook nog `postcss-loader` gebruiken om onze css verder te transformeren (autoprefixing, css modules, linting etc.). Er zijn veel verschillende loaders voor veel verschillende doeleinden.
Loaders bestaan voor de volgende (alle) transpilers die in dit onderzoek worden bekeken:

- Babel (babel-loader)
- SWC (swc-loader)
- TSC (ts-loader)
- Sucrase (@sucrase/webpack-loader, ?up-to-date?)
- ESBuild (esbuild-loader)

### Plugins

Webpack biedt ook heel wat plugins aan ([ook veel 3rd party](https://webpack.js.org/awesome-webpack/#webpack-plugins)). Plugins voor compression, typechecking, dependencies te auto-installeren etc.

### Modes

Zoals andere bundlers kan er ook gekozen worden voor een production of development build. In development builds zullen namen voor modules en chunks zinvol zijn, in production zijn ze mangled, en zijn bundles (eventueel, maar meestal) minified. Standaard zal in production gebuild worden, tenzij anders gevraagd.

### Compatibility

Webpack is compatibel met alle browsers die ES5-compliant zijn. ([In de praktijk is dat 99.54% van alle gebruikers online](https://caniuse.com/?search=es5))

## ESBuild

ESBuild is een bundler met ingebakken support voor TypeScript transpilatie.
ESBuild was gemaakt met performantie als nr. 1 goal. Het is geschreven in Go en de package die je download is natively compiled. Het gebruikt ook parallelization om nog meer perfomantie te bekomen. Het is gemaakt door de CTO van Figma, Evan Wallace.

De basisconfiguratie die Webpack aanbiedt is (op eerste oog) gelijkaardig. De configuratie kan hier ook in Go gebeuren, maar dat is minder relevant voor React-ontwikkeling. Een verschil is hier wel dat je ofwel via CLI, of via een [JS script te werk moet gaan](https://dev.to/marcinwosinek/how-to-configure-esbuild-with-a-build-script-2pcf). [Dit script is functioneel equivalent aan de Webpack.config.js file](https://blog.logrocket.com/getting-started-esbuild/).

Bij het analyseren van niet-statisch-analyseerbare imports gaat Webpack (onder andere) alle mogelijks bereikbare files toevoegen aan de bundle, en tijdens run-time een filesystem emuleren. Dit is en zal geen feature zijn in ESBuild. Note wel dat dit een vrij specifiek scenario is.

Standaard zal bij het builden ESBuild ES2020 targeten, tenzij anders geconfigureerd met de `--target` parameter.

ESBuild kan officieël ook maar tot ES6 builden, niet ES5 ([In de praktijk is dat 98.01% van alle gebruikers online](https://caniuse.com/?search=es6)).

Zoals hiervoor vernoemd kan ESBuild dus TypeScript transpileren. Er moet wel rekening gehouden worden met het feit dat ESBuild zelf geen type-checking doet, en je zelf in parallel nog `tsc -noEmit` zult moeten runnen. Hiernaast zijn er nog een aantal caveats in verband met [JS](https://esbuild.github.io/content-types/#javascript-caveats) en [TS](https://esbuild.github.io/content-types/#typescript-caveats).

JSX (en TSX) bestanden kunnen ook automatisch React importeren bij build indien ingesteld. ESBuild support voorlopig geen CSS modules of hot-module-replacement (HMR).
ESBuild kan JSON files bundelen naar JS modules. Tree shaking is altijd in gebruik, en code splitting is [nog niet af](https://esbuild.github.io/api/#splitting).

### Production readiness

ESBuild heeft nog geen 1.0.0 release en is in actieve development. Volgens hun FAQ zitten ze in een soort 'late-stage beta'. Of dit goed genoeg is voor Codifly gaat nog te bezien zijn.

### Main Take-away

ESBuild is first and foremost een bundler. Webpack en andere bundlers hebben de gewoonte van het bundlen te side-steppen. ESBuild zorgt voor performante en efficiënte builds. Zeker in grotere codebases gaat dit een enorm verschil maken. Het duurt wel even om te configureren, en de [bundels zijn meestal een beetje groter dan Rollup of Terser.](https://css-tricks.com/comparing-the-new-generation-of-build-tools)

## Vite

Vite is gemaakt door Evan You, de maker van Vue.js en is een volledige development server en optimised build tool tegelijk. Het is batteries-included, en werkt vaak zonder out-of-the-box. Het is echter wel een opinionated tool, het gebruikt Rollup voor bundling en ESBuild voor transpilatie en ondersteund HMR voor React-applicaties. Ook server-side rendering is een (momenteel experimentele) feature voor SPA frameworks.

### CSS

Vite heeft goede support voor css files. Bundelen van CSS imports, CSS modules, preprocessors, css code-splitting en het gebruik van PostCSS.

### Production Builds

Production builds gebeuren adhv een vooraf-geconfigureerde en -geoptimaliseerde Rollup-configuratie:

- Bundling
- Minification
- Tree-shaking
- code-splitting dynamic imports
- en meer...

## [Parcel](https://levelup.gitconnected.com/parcel-vs-webpack-2021-64c347bcb31)

Parcel is ook advertised als een no-configuration bundler.

### Features en notes

- Code-splitting en dynamic imports...
  - *maar* het bundelt alles samen een vlakke structuur. Webpack maakt folders aan voor images, CSS, JS etc.
- Parcel is trager dan Webpack op first run, maar is sneller bij subsequente builds (vooral bij het watchen).
- Hot Module Replacement
- Gebruikt SWC voor transpilatie
- Tree shaking
- Minification
- Image optimization
- Compression
- Content hashing
- Support Babel en PostCSS
- Plugin support

|                                | Webpack | ESBuild | Vite | Parcel | Rollup |
|:------------------------------:|---------|---------|------|--------|--------|
| Code-splitting                 | :heavy_check_mark: | WIP | :heavy_check_mark: | :heavy_check_mark: |        |
| Zero-configuration             | :heavy_check_mark:<br />(vanaf v5) | :x: | :heavy_check_mark: | :heavy_check_mark: |        |
| Plugin support                 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:<br />(compatibel met Rollup plugins) | :heavy_check_mark: | :heavy_check_mark: |
| Relatieve performantie (estimate) | Baseline | Tot 100x sneller | Gebruikt ESBuild | Trager op initiële build, tot 30% sneller op subsequente builds. | Tot 20% sneller |
| Customisability                | Hoog | Hoog | Laag | Laag | Hoog |
| Dynamic expressionx in import()| :heavy_check_mark:<br />(base directory moet gekend zijn) | :x:<br />(workaround bestaat) | :heavy_check_mark:<br />(met de [@rollup/plugin-dynamic-import-vars](https://github.com/rollup/plugins/tree/master/packages/dynamic-import-vars) plugin) | :x:<br />(wel wildcard imports) | :heavy_check_mark:<br />(met de [@rollup/plugin-dynamic-import-vars](https://github.com/rollup/plugins/tree/master/packages/dynamic-import-vars) plugin) |
| Laagste ECMA target            | ES5 | ES6 | ES5 | ES5 | ES5 |
| Support voor TS transpilers in deze paper | Alle | Eigen transpiler | Alle adhv Rollup plugins, standaard ESBuild | TSC, Babel, SWC (default) | Alle adhv Rollup plugins, standaard ESBuild |
| TypeScript typechecking        | :heavy_check_mark:<br />(plugin) | :x: | :heavy_check_mark:<br />(Rollup plugin) | :x:<br />(experimentele plugin) | :heavy_check_mark:<br />(plugin) |
| HMR (Hot module replacement)   | :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark:<br />(plugin) |
| Tree shaking                   | :heavy_check_mark:<br />(customisable) | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Serverside rendering support   | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| CSS features                   | Alles (met plugins) | Alles behalve code-splitting (met plugins) | Alles (ingebouwd) | Alles behalve code-splitting | Alles (met plugins) |
| Compression dev server | gzip, brotli | Unofficial plugin | brotli (gzip met unofficial plugin) | gzip, brotli | gzip, brotli |

https://stackoverflow.com/a/32172835/10780174
https://www.linkedin.com/pulse/why-do-only-3-top-1000-websites-use-http2-server-push-samir-jafferali/

http2
Indivi:

- compression minder goed:
origineel bestand:
<img src="img/compression-1.jpg" width="400"/>
losless compression:
<img src="img/compression-2.jpg" width="400"/>
- als er een diff is op live vs cached dan moeten alleen de changed modules opgehaald worden

Bundle:

- compression beter (see images above)
- als er een diff is op live vs cached dan moet de hele bundel opnieuw worden opgehaald

## Webpack


