# Angular Performance Checklist

<img src="./assets/flash.png" width="1000">

- [中文版](./README.zh-CN.md)
- [Русский](./README.ru-RU.md)
- [Português](./README.pt-BR.md)
- [Español](./README.es-ES.md)
- [日本語](./README.ja-JP.md)
- [Türkçe](./README.tr-TR.md)

## Giriş

Bu döküman Angular uygulamalarımızın performansını arttırmamıza yardımcı olacak yöntemleri içermektedir. "Angular Performance Checklist", uygulamalarımızın server side rendering pre-rendering ve paketlenmesinden runtime performansına ve change detection optimizasyonuna kadar konuları kapsamaktadır.

Bu döküman iki ana bölümden oluşmaktadır:

- Network Performansı
- Runtime Performansı

- Network performance - uygulamalarımızın yüklenme süresini iyileştirecek yöntemleri içermektedir. Latency ve bandwidth azaltma yöntemlerini de içermektedir.
- Runtime performance - uygulamanın runtime performansını iyileştirecek yöntemleri içermektedir. Çoğunlukla change detection ve rendering optimizasyonları ile ilgilidir.

Bazı yöntemler her iki kategoriye de girmektedir, ancak kullanım durumları ve sonuçlarındaki farklılıklar açıkça belirtilecektir.

Çoğu alt bölüm, geliştirme akışımızı otomatikleştirecek ve bizi daha verimli hale getirebilecek uygulamayla ilgili araçları listelemektedir.

Çoğu yöntemin hem HTTP/1.1 hem de HTTP/2 için geçerli olduğunu unutmayın. İstisna olan durumlarda, protokolün hangi versiyonuna uygulanabileceği belirtilecektir.

## İçerik

- [Angular Performance Checklist](#angular-2-performance-checklist)
    - [Giriş](#giriş)
    - [İçerik](#İçerik)
    - [Network performansı](#network-performansı)
        - [Bundling](#bundling)
        - [Minification and Dead code elimination](#minification-and-dead-code-elimination)
        - [Remove template whitespace](#remove-template-whitespace)
        - [Tree-shaking](#tree-shaking)
        - [Tree-shakeable providers](#tree-shakeable-providers)
        - [Ahead-of-Time (AoT) Compilation](#ahead-of-time-aot-compilation)
        - [Compression](#compression)
        - [Pre-fetching Resources](#pre-fetching-resources)
        - [Lazy-Loading of Resources](#lazy-loading-of-resources)
        - [Don't lazy-load default route](#dont-lazy-load-the-default-route)
        - [Caching](#caching)
        - [Use Application Shell](#use-application-shell)
        - [Use Service Workers](#use-service-workers)
    - [Runtime Optimizations](#runtime-optimizations)
        - [Use `enableProdMode`](#use-enableprodmode)
        - [Ahead-of-Time Compilation](#ahead-of-time-compilation)
        - [Web Workers](#web-workers)
        - [Server-Side Rendering](#server-side-rendering)
        - [Change Detection](#change-detection)
            - [`ChangeDetectionStrategy.OnPush`](#changedetectionstrategyonpush)
            - [Detaching the Change Detector](#detaching-the-change-detector)
            - [Run outside Angular](#run-outside-angular)
            - [Coalescing event change detections](#coalescing-event-change-detections)
        - [Use pure pipes](#use-pure-pipes)
        - [`*ngFor` directive](#ngfor-directive)
            - [Use `trackBy` option](#use-trackby-option)
            - [Minimize DOM elements](#minimize-dom-elements)
        - [Optimize template expressions](#optimize-template-expressions)
- [Conclusion](#conclusion)
- [Contributing](#contributing)

## Network performansı

Bu bölümdeki yöntemlerden bazıları hala geliştirme aşamasındadır ve değişebilmektedir. Angular ekibi, pek çok şeyin açık şekilde gerçekleşmesi için uygulama oluşturma sürecini mümkün olduğunca otomatize etmeye çalışıyor.

### Bundling

Paketleme kullanıcının istekte bulunduğu bir uygulamaya erişebilmesi için tarayıcının yapcağı istek sayısını azaltmayı amaçlayan standart bir uygulamadır. Aslında paketleyici girdi olarak birden fazla giriş noktaları alır ve bir veya daha fazla paket üretir. Bu şekilde tarayıcı, her bir kaynağı ayrı ayrı istemek yerine, yalnızca birkaç istek gerçekleştirerek uygulamanın tamamını alabilir.

Uygulamanın boyutu büyüdükçe her şeyi tek bir pakette toplamak da uygulamanız için bir dezavantaj olacaktır. Bu yüzden Webpack kullanarak Code Splitting tekniklerini kullanmak gerekmektedir.

**Ek http istekleri will HTTP/2'nin [server push](https://http2.github.io/faq/#whats-the-benefit-of-server-push) özelliğinden dolayı sorun olmayacaktır.**

**Tooling**

Uygulamalarımızı en verimli şekilde paketlememizi sağlayan araçlar şunlardır:

- [Webpack](https://webpack.js.org) - [tree-shaking](#tree-shaking) gerçekleştirerek paketlemeyi sağlar.
- [Webpack Code Splitting](https://webpack.js.org/guides/code-splitting/) - Kodu farklı bölümlere ayırmayı sağlar.
- [Webpack & http2](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.46idrz8kb) - http2 ile birlikte kod bölümlemeyi sağlar.
- [Rollup](https://github.com/rollup/rollup) - ES2015 modüllerinin statik yapısından yararlanarak tree-shaking kullarak paketlemeyi sağlar.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - Çok sayıda optimizasyon gerçekleştirir ve paketleme desteği sağlar. Orijinal olarak Java'da yazılmıştır. JavaScript versiyon'una [buradan erişilebilir.](https://www.npmjs.com/package/google-closure-compiler).
- [SystemJS Builder](https://github.com/systemjs/builder) - provides a single-file build for SystemJS of mixed-dependency module trees.
- [SystemJS Builder](https://github.com/systemjs/builder) - Karışık bağımlığı olan SystemJS için tekli dosya oluşturmayı sağplar.
- [Browserify](http://browserify.org/).

**Kaynaklar**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Küçültme ve ölü kodun kaldırılması

Bu yöntemler, uygulamamızın yükünü azaltarak bant genişliği tüketimini en aza indirmemizi sağlar.

**Tooling**

- [Uglify](https://github.com/mishoo/UglifyJS) - değişkenleri yönetme, yorumların ve boşlukların kaldırılması, ölü kodların ortadan kaldırılması vb. gibi küçültme işlemlerini gerçekleştirir. Tamamen JavaScript ile yazılmıştır, tüm popüler görev yürütücüler için eklentilere sahiptir.
-[Google Closure Compiler](https://github.com/google/closure-compiler) - "Uglify" küçültme yöntemlerine benzer çalışmaktadır. Gelişmiş modda uygulamamızın AST(Absctrack Syntax Tree)'sini çok daha karmaşık optimizasyonlarda bile dönüştürmek için kullanılır. Ayrıca [JavaScript versiyonu](https://www.npmjs.com/package/google-closure-compiler) [buradan bulunabilir.](https://www.npmjs.com/package/google-closure-compiler). Bunun yanında GCC *çoğu ES2015 modül sözdizimini* desteklemektedir. Bu yüzden [tree-shaking](#tree-shaking) de uygulayabilmektedir.
  
**Kaynaklar**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Şablon Boşluklarının Kaldırılması

Boşluk karakterlerini ('\s' ifadesiyle eşleşen bir karakter) görmememize rağmen, yine de ağ üzerinden aktarılan baytlarla temsil edilir. Şablonlarımızdaki boşlukları minimuma indirmek, AOT kodumuzun paketlenmiş boyutunu daha da azaltılmış olacaktır.
Neyse ki bunu manual olarak yapmak durumunda değiliz. `ComponentMetadata` interface'inde bulunan `preserveWhitespaces` default olarak `false` değerini alarak uygulama içerisindeki boşluk ifadelerini Angular derleyicisi kaldıracaktır. Eğer `preserveWhitespace` özelliği `true` olarak düzenlenirse boşluk ifadeleri korunacaktır.
- [Angular dökümanı: preserveWhitespaces](https://angular.io/api/core/Component#preserveWhitespaces)

### Tree-shaking
Uygulamalarımızın son sürümü için, genellikle Angular ve/veya herhangi bir üçüncü taraf kütüphanesi tarafından oluşturulmuş kodun tamamı, geliştiricinin yazmuş olduğu kısımlar bile düşünüldüğünde tamamen kullanılmamaktadır. ES2015 modüllerinin statik yapısı sayesinde uygulamalarımızda referans verilmeyen kodlardan kurtulabiliyoruz.

**Örnek**

```javascript
// foo.js
export foo = () => 'foo';
export bar = () => 'bar';

// app.js
import { foo } from './foo';
console.log(foo());
```

tree-shake uygulandığında ve paketlendiğinde `app.js` dosyası şu şekilde olacaktır:

```javascript
let foo = () => 'foo';
console.log(foo());
```

Bunun anlamı kullanılmayan ve export edilmiş `bar` fonksiyonu pakete dahil edilmeyecektir.

**Tooling**

- [Webpack](https://webpack.js.org) - [tree-shaking](#tree-shaking) ile verimli bir paketlemeyi sağlamaktadır. Uygulama paketlendikten sonra, kullanılmayan kodu dışa aktarmaz, böylece kolayca ölü kod olarak kabul edilebilir ve Uglify tarafından kaldırılabilir.
- [Rollup](https://github.com/rollup/rollup) - ES2015 modüllerinin statik yapısından yararlanarak verimli bir tree-shaking gerçekleştirir.
- [Google Closure Compiler](https://github.com/google/closure-compiler) - çok sayıda optimizasyon gerçekleştirir ve paketleme desteği sağlar. Java ile yazılmıştır, bunun yanında [JavaScript versiyonu](https://www.npmjs.com/package/google-closure-compiler) da [bulunmaktadır](https://www.npmjs.com/package/google-closure-compiler).

*Not:* GCC, `barrel` kalıbının yoğun kullanımı nedeniyle Angular uygulamaları oluşturmak için gerekli olan `export *` henüz desteklenmemektedir.

**Kaynaklar**

- ["Building an Angular Application for Production"](http://blog.mgechev.com/2016/06/26/tree-shaking-angular2-production-build-rollup-javascript/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
- ["Using pipeable operators in RxJS"](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)

### Tree-Shakeable Providers

Angular 6 versiyonu ile birlikte, Angular geliştirici takımı servislerin tree-shakeable olabileceği bir özellik yayınladı. Böylece bir component veya bir servis tarafından kullanılmayan servisler uygulama paketine dahil edilmeyerek paket boyutunun azaltılmasına yardımcı olacaktır.

Servisler oluşturulurken `@Injectable()` decorator'u ile birlikte `provideIn` özelliği kullanılarak nerede başlatılacağını belirtilebilir ve tree-shakeable hale getirilebilir. Bu şekilde oluşturulan servisler NgModule içerisinden aşağıdaki gibi kaldırılabilir.
Öncesi:

```ts
// app.module.ts
import { NgModule } from '@angular/core'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'
import { environment } from '../environments/environment'
import { MyService } from './app.service'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    ...
  ],
  providers: [MyService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// my-service.service.ts
import { Injectable } from '@angular/core'

@Injectable()
export class MyService { }
```

Sonrası:

```ts
// app.module.ts
import { NgModule } from '@angular/core'
import { AppRoutingModule } from './app-routing.module'
import { AppComponent } from './app.component'
import { environment } from '../environments/environment'

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    ...
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// my-service.service.ts
import { Injectable } from '@angular/core'

@Injectable({
  providedIn: 'root'
})
export class MyService { }
```
`MyService` herhangi component veya serviste kullanılmadığı durumda pakete dahil edilmeyecektir.

**Kaynaklar**

- [Angular Providers](https://angular.io/guide/providers)

### Zamanından Önce Derleme (AOT - Ahead Of Time Compilation)

A challenge for the available in the wild tools (such as GCC, Rollup, etc.) are the HTML-like templates of the Angular components, which cannot be analyzed with their capabilities. This makes their tree-shaking support less efficient because they're not sure which directives are referenced within the templates. The AoT compiler transpiles the Angular HTML-like templates to JavaScript or TypeScript with ES2015 module imports. This way we are able to efficiently tree-shake during bundling and remove all the unused directives defined by Angular, third-party libraries or by ourselves.

**Kaynaklar**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Sıkıştırma

Bant genişliği kulanımını azaltmak için yanıtların yük standartının sıkıştırılması için bazı konfigürasyonlar yapılabilir. `Accept-Encoding` başlığına bir değer verilerek tarayıcı tarafında kullanıcının hangi sıkıştırma algoritmalarının kullanılabileceği konsunda sunucu tarafına bilgi verebilir. Diğer yandan sunucu, bir isteğe cevap verirken, cevabın hangi sıkıştırılması için hangi algoritma kullanıldığını tarayıcıya `Content-Encoding` başlık değerinde belirtmektedir.

**Tooling**

Buradaki araçlar Angular'a özgü değildir ve tamamen kullandığımız web/uygulama sunucusuna bağlıdır. Tipik sıkıştırma algoritmaları şunlardır:

- deflate - LZ77 algoritması ve Huffman kodlamasının bir kombinasyonunu kullanan bir veri sıkıştırma algoritması ve ilişkili dosya formatıdır.
- [brotli](https://github.com/google/brotli) - LZ77 algoritmasının modern bir varyantı, Huffman kodlaması ve 2. dereceden bağlam modellemesinin bir kombinasyonunu kullanarak verileri şu anda mevcut olan en iyi genel amaçlı sıkıştırma yöntemleriyle karşılaştırılabilir bir sıkıştırma oranıyla sıkıştıran genel amaçlı kayıpsız bir sıkıştırma algoritması. Hız olarak deflate ile benzer fakat daha fazla sıkıştırma sağlamaktadır.
  

  **Kaynaklar**

- ["Better than Gzip Compression with Brotli"](https://hacks.mozilla.org/2015/11/better-than-gzip-compression-with-brotli/)
- ["2.5X Smaller Angular Applications with Google Closure Compiler"](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)

### Kaynakları Önden Getirme (Pre-fetching resources)

Kaynakların önceden getirilmesi kullanıcı deneyimini arttırmak için iyi bir yöntemdir. Bunun yanında varlıkları assets ([tembel (lazy)](#lazy-loading-of-resources), yüklenmesi amaçlanan resimler, modüller vs.) ve verileri de önden getirebiliriz. Farklı önden getirme yöntemleri vardır, ancak bunların çoğu uygulamanın özelliklerine bağlıdır.

### Lazy-Loading of Resources

In case the target application has a huge code base with hundreds of dependencies, the practices listed above may not help us reduce the bundle to a reasonable size (reasonable might be 100K or 2M, it again, completely depends on the business goals).

In such cases, a good solution might be to load some of the application's modules lazily. For instance, let's suppose we're building an e-commerce system. In this case, we might want to load the admin panel independently from the user-facing UI. Once the administrator has to add a new product we'd want to provide the UI required for that. This could be either only the "Add product page" or the entire admin panel, depending on our use case/business requirements.

**Tooling**

- [Webpack](https://github.com/webpack/webpack) - allows asynchronous module loading.
- [ngx-quicklink](https://github.com/mgechev/ngx-quicklink) - router preloading strategy which automatically downloads the lazy-loaded modules associated with all the visible links on the screen

### Don't Lazy-Load the Default Route

Let's suppose we have the following routing configuration:

```ts
// Bad practice
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard',  loadChildren: () => import('./dashboard.module').then(mod => mod.DashboardModule) },
  { path: 'heroes', loadChildren: () => import('./heroes.module').then(mod => mod.HeroesModule) }
];
```

The first time the user opens the application using the url: https://example.com/ they will be redirected to `/dashboard` which will trigger the lazy-route with path `dashboard`. In order Angular to render the bootstrap component of the module, it will has to download the file `dashboard.module` and all of its dependencies. Later, the file needs to be parsed by the JavaScript VM and evaluated.

Triggering extra HTTP requests and performing unnecessary computations during the initial page load is a bad practice since it slows down the initial page rendering. Consider declaring the default page route as non-lazy.

### Caching

Caching is another common practice intending to speed-up our application by taking advantage of the heuristic that if one resource was recently been requested, it might be requested again in the near future.

For caching data we usually use a custom caching mechanism. For caching static assets, we can either use the standard browser caching or Service Workers with the [CacheStorage API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).

### Use Application Shell

To make the perceived performance of your application faster, use an [Application Shell](https://developers.google.com/web/updates/2015/11/app-shell).

The application shell is the minimum user interface that we show to the users in order to indicate them that the application will be delivered soon. For generating an application shell dynamically you can use Angular Universal with custom directives which conditionally show elements depending on the used rendering platform (i.e. hide everything except the App Shell when using `platform-server`).

**Tooling**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (isomorphic) JavaScript support for Angular.

**Resources**

- ["Instant Loading Web Apps with an Application Shell Architecture"](https://developers.google.com/web/updates/2015/11/app-shell)

### Use Service Workers

We can think of the Service Worker as an HTTP proxy which is located in the browser. All requests sent from the client are first intercepted by the Service Worker which can either handle them or pass them through the network.

You can add a Service Worker to your Angular project by running
``` ng add @angular/pwa ```

**Tooling**

- [Angular Service Worker](https://angular.io/guide/service-worker-intro) - aims to automate the process of managing Service Workers. It also contains Service Worker for caching static assets and one for [generating application shell](https://developers.google.com/web/updates/2015/11/app-shell?hl=en).
- [Offline Plugin for Webpack](https://github.com/NekR/offline-plugin) - Webpack plugin that adds support for Service Worker with a fall-back to AppCache.

**Resources**

- ["The offline cookbook"](https://jakearchibald.com/2014/offline-cookbook/)
- ["Getting started with service workers"](https://angular.io/guide/service-worker-getting-started)

## Runtime Optimizations

This section includes practices that can be applied in order to provide smoother user experience with 60 frames per second (fps).

### Use `enableProdMode`

In development mode, Angular performs some extra checks in order to verify that performing change detection does not result in any additional changes to any of the bindings. This way the frameworks assures that the unidirectional data flow has been followed.

In order to disable these changes for production do not forget to invoke `enableProdMode`:

```typescript
import { enableProdMode } from '@angular/core';

if (ENV === 'production') {
  enableProdMode();
}
```

### Ahead-of-Time Compilation

AoT can be helpful not only for achieving more efficient bundling by performing tree-shaking, but also for improving the runtime performance of our applications. The alternative of AoT is Just-in-Time compilation (JiT) which is performed runtime, therefore we can reduce the amount of computations required for the rendering of our application by performing the compilation as part of our build process.

**Tooling**

- [angular2-seed](https://github.com/mgechev/angular2-seed) - a starter project which includes support for AoT compilation.
- [angular-cli](https://cli.angular.io) Using the `ng serve --prod`

**Resources**

- ["Ahead-of-Time Compilation in Angular"](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)

### Web Workers

The usual problem in the typical single-page application (SPA) is that our code is usually run in a single thread. This means that if we want to achieve smooth user experience with 60fps we have **at most 16ms** for execution between the individual frames are being rendered, otherwise, they'll drop by half.

In complex applications with a huge component tree, where the change detection needs to perform millions of checks each second it will not be hard to start dropping frames. Thanks to the Angular's agnosticism and being decoupled from DOM architecture, it's possible to run our entire application (including change detection) in a Web Worker and leave the main UI thread responsible only for rendering.

**Tooling**

- The module which allows us to run our application in a Web Worker is supported by the core team. Examples of how it can be used can be [found here](https://github.com/angular/angular/tree/master/modules/playground/src/web_workers).
- [Webpack Web Worker Loader](https://github.com/webpack/worker-loader) - A Web Worker Loader for webpack.

**Resources**

- ["Using Web Workers for more responsive apps"](https://www.youtube.com/watch?v=Kz_zKXiNGSE)

### Server-Side Rendering

A big issue of the traditional SPA is that they cannot be rendered until the entire JavaScript required for their initial rendering is available. This leads to two big problems:

- Not all search engines are running the JavaScript associated with the page so they are not able to index the content of dynamic apps properly.
- Poor user experience, since the user will see nothing more than a blank/loading screen until the JavaScript associated with the page is downloaded, parsed and executed.

Server-side rendering solves this issue by pre-rendering the requested page on the server and providing the markup of the rendered page during the initial page load.

**Tooling**

- [Angular Universal](https://github.com/angular/angular/tree/master/packages/platform-server) - Universal (isomorphic) JavaScript support for Angular.
- [Preboot](https://github.com/angular/preboot) - Library to help manage the transition of state (i.e. events, focus, data) from a server-generated web view to a client-generated web view.
- [Scully](https://github.com/scullyio/scully) - Static site generator for Angular projects looking to embrace the JAMStack.

**Resources**

- ["Angular Server Rendering"](https://www.youtube.com/watch?v=0wvZ7gakqV4)
- ["Angular Universal Patterns"](https://www.youtube.com/watch?v=TCj_oC3m6_U)
- ["Create a Static Site Using Angular & Scully"](https://www.youtube.com/watch?v=ugTx-14jRrI)

### Change Detection

On each asynchronous event, Angular performs change detection over the entire component tree. Although the code which detects for changes is optimized for [inline-caching](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html), this still can be a heavy computation in complex applications. A way to improve the performance of the change detection is to not perform it for subtrees which are not supposed to be changed based on the recent actions.

#### `ChangeDetectionStrategy.OnPush`

The `OnPush` change detection strategy allows us to disable the change detection mechanism for subtrees of the component tree. By setting the change detection strategy to any component to the value `ChangeDetectionStrategy.OnPush` will make the change detection perform **only** when the component has received different inputs. Angular will consider inputs as different when it compares them with the previous inputs by reference, and the result of the reference check is `false`. In combination with [immutable data structures](https://facebook.github.io/immutable-js/), `OnPush` can bring great performance implications for such "pure" components.

**Resources**

- ["Change Detection in Angular"](https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c)
- ["Everything you need to know about change detection in Angular"](https://blog.angularindepth.com/everything-you-need-to-know-about-change-detection-in-angular-8006c51d206f)

#### Detaching the Change Detector

Another way of implementing a custom change detection mechanism is by `detach`ing and `reattach`ing the change detector (CD) for given a component. Once we `detach` the CD Angular will not perform check for the entire component subtree.

This practice is typically used when user actions or interactions with external services trigger the change detection more often than required. In such cases we may want to consider detaching the change detector and reattaching it only when performing change detection is required.

#### Run outside Angular

The Angular's change detection mechanism is being triggered thanks to [zone.js](https://github.com/angular/zone.js). Zone.js monkey patches all asynchronous APIs in the browser and triggers the change detection at the end of the execution of any async callback. In **rare cases**, we may want the given code to be executed outside the context of the Angular Zone and thus, without running change detection mechanism. In such cases, we can use the method `runOutsideAngular` of the `NgZone` instance.

**Example**

In the snippet below, you can see an example for a component that uses this practice. When the `_incrementPoints` method is called the component will start incrementing the `_points` property every 10ms (by default). The incrementation will make the illusion of an animation. Since in this case, we don't want to trigger the change detection mechanism for the entire component tree, every 10ms, we can run `_incrementPoints` outside the context of the Angular's zone and update the DOM manually (see the `points` setter).

```ts
@Component({
  template: '<span #label></span>'
})
class PointAnimationComponent {

  @Input() duration = 1000;
  @Input() stepDuration = 10;
  @ViewChild('label') label: ElementRef;

  @Input() set points(val: number) {
    this._points = val;
    if (this.label) {
      this.label.nativeElement.innerText = this._pipe.transform(this.points, '1.0-0');
    }
  }
  get points() {
    return this._points;
  }

  private _incrementInterval: any;
  private _points: number = 0;

  constructor(private _zone: NgZone, private _pipe: DecimalPipe) {}

  ngOnChanges(changes: any) {
    const change = changes.points;
    if (!change) {
      return;
    }
    if (typeof change.previousValue !== 'number') {
      this.points = change.currentValue;
    } else {
      this.points = change.previousValue;
      this._ngZone.runOutsideAngular(() => {
        this._incrementPoints(change.currentValue);
      });
    }
  }

  private _incrementPoints(newVal: number) {
    const diff = newVal - this.points;
    const step = this.stepDuration * (diff / this.duration);
    const initialPoints = this.points;
    this._incrementInterval = setInterval(() => {
      let nextPoints = Math.ceil(initialPoints + diff);
      if (this.points >= nextPoints) {
        this.points = initialPoints + diff;
        clearInterval(this._incrementInterval);
      } else {
        this.points += step;
      }
    }, this.stepDuration);
  }
}
```

**Warning**: Use this practice **very carefully only when you're sure what you are doing** because if not used properly it can lead to an inconsistent state of the DOM. Also, note that the code above is not going to run in WebWorkers. In order to make it WebWorker-compatible, you need to set the label's value by using the Angular's renderer.

#### Coalescing event change detections

Angular uses zone.js to intercept events that occurred in the application and runs a change detection automatically. By default this happens when the [microtask queue](https://www.youtube.com/watch?v=cCOL7MC4Pl0) of the browser is empty, which in some cases may call redundant cycles.
From v9, Angular provides a way to coalesce event change detections by turning `ngZoneEventCoalescing` on, i.e
```typescript
platformBrowser()
  .bootstrapModule(AppModule, { ngZoneEventCoalescing: true });
```
The above configuration will schedule change detection with `requestAnimationFrame`, instead of plugging into the microtask queue, which will run checks less frequently and consume fewer computational cycles.

> Warning: **ngZoneEventCoalescing: true** may break existing apps that relay on consistently running change detection.


**Resources**
- [ngZoneEventCoalescing BootstrapOption](https://github.com/angular/angular/blob/master/packages/core/src/application_ref.ts#L268) - source code for BootstrapOptions interface
- [Reduce Change Detection Cycles with Event Coalescing in Angular](https://netbasal.com/reduce-change-detection-cycles-with-event-coalescing-in-angular-c4037199859f)
- [Simple Angular context help component or how global event listener can affect your perfomance](https://medium.com/@a.yurich.zuev/simple-angular-context-help-component-or-how-global-event-listener-can-affect-your-perfomance-75b67dba197f)

### Use pure pipes

As argument the `@Pipe` decorator accepts an object literal with the following format:

```typescript
interface PipeMetadata {
  name: string;
  pure: boolean;
}
```

The pure flag indicates that the pipe is not dependent on any global state and does not produce side-effects. This means that the pipe will return the same output when invoked with the same input. This way Angular can cache the outputs for all the input parameters the pipe has been invoked with, and reuse them in order to not have to recompute them on each evaluation.

The default value of the `pure` property is `true`.

### `*ngFor` directive

The `*ngFor` directive is used for rendering a collection.

#### Use `trackBy` option

By default `*ngFor` identifies object uniqueness by reference.

Which means when a developer breaks reference to object during updating item's content Angular treats it as removal of the old object and addition of the new object. This effects in destroying old DOM node in the list and adding new DOM node on its place.

The developer can provide a hint for angular how to identify object uniqueness: custom tracking function as the `trackBy` option for the `*ngFor` directive. The tracking function takes two arguments: `index` and `item`. Angular uses the value returned from the tracking function to track items identity. It is very common to use the ID of the particular record as the unique key.

**Example**

```typescript
@Component({
  selector: 'yt-feed',
  template: `
  <h1>Your video feed</h1>
  <yt-player *ngFor="let video of feed; trackBy: trackById" [video]="video"></yt-player>
`
})
export class YtFeedComponent {
  feed = [
    {
      id: 3849, // note "id" field, we refer to it in "trackById" function
      title: "Angular in 60 minutes",
      url: "http://youtube.com/ng2-in-60-min",
      likes: "29345"
    },
    // ...
  ];

  trackById(index, item) {
    return item.id;
  }
}
```

#### Minimize DOM elements

Rendering the DOM elements is usually the most expensive operation when adding elements to the UI. The main work is usually caused by inserting the element into the DOM and applying the styles. If `*ngFor` renders a lot of elements, browsers (especially older ones) may slow down and need more time to finish rendering of all elements. This is not specific to Angular.

To reduce rendering time, try the following:
- Apply virtual scrolling via [CDK](https://material.angular.io/cdk/scrolling/overview) or [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller)
- Reducing the amount of DOM elements rendered in `*ngFor` section of your template. Usually, unneeded/unused DOM elements arise from extending the template again and again. Rethinking its structure probably makes things much easier.
- Use [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer) where possible

**Resources**

- ["NgFor directive"](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) - official documentation for `*ngFor`
- ["Angular — Improve performance with trackBy"](https://netbasal.com/angular-2-improve-performance-with-trackby-cc147b5104e5) - shows gif demonstration of the approach
- [Component Dev Kit (CDK) Virtual Scrolling](https://material.angular.io/cdk/scrolling/overview) - API description
- [ngx-virtual-scroller](https://github.com/rintoj/ngx-virtual-scroller) - displays a virtual, "infinite" list

### Optimize template expressions

Angular executes template expressions after every change detection cycle. Change detection cycles are triggered by many asynchronous activities such as promise resolutions, http results, timer events, keypresses, and mouse moves.

Expressions should finish quickly or the user experience may drag, especially on slower devices. Consider caching values when their computation is expensive.

**Resources**
- [quick-execution](https://angular.io/guide/template-syntax#quick-execution) - official documentation for template expressions
- [Increasing Performance - more than a pipe dream](https://youtu.be/I6ZvpdRM1eQ) - ng-conf video on youtube. Using pipe instead of function in interpolation expression

# Conclusion

The list of practices will dynamically evolve over time with new/updated practices. In case you notice something missing or you think that any of the practices can be improved do not hesitate to fire an issue and/or a PR. For more information please take a look at the "[Contributing](#contributing)" section below.

# Contributing

In case you notice something missing, incomplete or incorrect, a pull request will be greatly appreciated. For discussion of practices that are not included in the document please [open an issue](https://github.com/mgechev/angular2-performance-checklist/issues).

# License

MIT
