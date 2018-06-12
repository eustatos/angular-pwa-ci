
Нам необоходим angular-cli версии не ниже 6.0.0, поэтому проверим версию установленного пакета:
```bash
ng --version
```
и при необходимости установим последнюю версию:
```bash
npm i -g angular-cli
```
Все это мы проделываем, так как существуют различия в структуре проектов, которые создаются angular-cli 6 версии и проектов созданными более ранними версиями. Так же имеются различия и в функциональности. Как раз о некоторых из этих особенностей эта статья. 

Итак, создадим типовой проект сразу с функционностью роутинга, так как планируется реализовать и демонстрацию возможостей отложенной загрузки модулей (lazy loading modules):
```bash
ng new angular-pwa-ci --routing
```
Чтобы удобнее было отслеживать изменения, которые будут происходить с проектом сделаем первый коммит.
```bash
git init
git add .
git commit -m "Initial commit"
```
Следующая команда превратит наш проект Progressive Web Application (PWA)
```bash
ng add @angular/pwa --project "angular-pwa-ci"
```
Выдача по результатам выполнения команды содержит следующее:
```
Installed packages for tooling via yarn.
CREATE ngsw-config.json (392 bytes)
CREATE src/assets/icons/icon-128x128.png (1253 bytes)
CREATE src/assets/icons/icon-144x144.png (1394 bytes)
CREATE src/assets/icons/icon-152x152.png (1427 bytes)
CREATE src/assets/icons/icon-192x192.png (1790 bytes)
CREATE src/assets/icons/icon-384x384.png (3557 bytes)
CREATE src/assets/icons/icon-512x512.png (5008 bytes)
CREATE src/assets/icons/icon-72x72.png (792 bytes)
CREATE src/assets/icons/icon-96x96.png (958 bytes)
CREATE src/manifest.json (1085 bytes)
UPDATE angular.json (3571 bytes)
UPDATE package.json (1389 bytes)
UPDATE src/app/app.module.ts (605 bytes)
UPDATE src/index.html (390 bytes)
```
Сделаем еще один коммит.
И поспешим убедиться насколько наше приложение теперь соответствует требованиям, предъявляемым к PWA.
Для этого воспользуемся утилитой `lighthouse`, которая проведет аудит нашего приложения и сгенерирует подробный отчет по его результатам.
Если эта утилита еще не установлена, то установить ее можно командой:
```bash
npm i -g lighthouse
```
Эта утилита, в том числе будет проверять отображение нашим приложением контента при отключенном в браузере javascript. Поэтому проверим наличие в файле `src/index.html` строки
```html
  <noscript>Please enable JavaScript to continue using this application.</noscript>
```

Теперь выполним сборку нашего проекта в режиме `production`, так как настойками по умолчанию предусмотрена работа service workers только в режиме `prodaction`.
```bash
ng build --prod
```
Далее в данной статье будет описана процедура auto-deploy на бесплатный хостинг предоставляемый сервисом firebase, но сейчас для целей разработки нам будет достаточно локального сервера, который мы сейчас и напишем. Создадим в корне проекта файл `server.js`
Если вы используете в качестве редактора Visual Studio Code, то рекомендую установить расширение [ Angular Essential ]( https://marketplace.visualstudio.com/items?itemName=johnpapa.angular-essentials ), которой включает в себя расширение [ Angular v6 Snippets ]( https://marketplace.visualstudio.com/items?itemName=johnpapa.Angular2 ), с помощью которого можно начать вводить:
```
ex-node-
```
![](https://habrastorage.org/webt/qq/nx/px/qqnxpxy1fg2x543ojpwrypsyasc.gif)
Согласится с предложенным сниппетом, указать желаемый порт и папку, в которой находятся файлы для отображения. И все. Можно запускать:
```bash
node server.js
```
Наше приложение доступно по адресу: `http://localhost:8080` и мы можем запустить аудит
```bash
lighthouse http://localhost:8080
````
Утилита создаст в корне нашего проекта файл вида `localhost_2018-06-08_23-42-21.report.html`. Откроем его в браузере и увидим результаты аудита. 100% мы не набрали, но все еще впереди.

Для того чтобы организовать автодеплой нашего приложения на firebase, на понадобится аккаунт на http://firebase.com. 

После этого установим `firebase-tools`. Устанавливать будем локально, так как этот пакет будет использоваться в дальнейшем для автодеплоя.
```bash
npm i -D firebase-tools
```
А чтобы не писать длинный путь `node_models/firebase-tools/bin/firebase` каждый раз - установим еще и глобально.
```bash
npm i -g firebase-tools
```

Залогинимся в сервисе firebase:
```bash
firebase login
```
Эта команда вызовет открытие браузера по умолчанию на странице, где будет предложено дать разрешение приложению. Дадим согласие. Можно работать дальше.

Создание нового проекта возможно только в Firebase Console, поэтому перейдем по адресу `https://console.firebase.google.com` и создадим новый проект, который назовем `angular-pwa-ci`.

Следующией командой создадим файлы конфигурации.
```bash
./node_modules/firebase-tools/bin/firebase init
```
Эта команда вызовет диалог, где мы:
* выберем проект `angular-pwa-ci`; 
* из сервисов будем использовать только `hosting`;
* папкой для синхронизации укажем `dist/angular-pwa-ci/`;
* сконфигурируем наше приложение, как SPA (переадресуем все запросы на index.html);
* откажемся от перезаписи index.html.

Теперь выложим наше приложение на хостинг в ручном режиме
```bash
./node_modules/firebase-tools/bin/firebase deploy --only hosting
```
В выдаче этой команды, будет указан адрес, по которому будет доступно наше приложение. Например, `https://angular-pwa-ci.firebaseapp.com`.

А теперь еще раз проведем аудит нашего приложения.
```bash
lighthouse https://angular-pwa-ci.firebaseapp.com
```

![](https://habrastorage.org/webt/wg/sn/qv/wgsnqvw-8vqssjbvajlxef_f8vc.png)

Вот наше прилоение стало PWA на 100%.

## Lazy loading modules

Немного украсим наше приложение. Заодно исследуем еще одну из возможностей angular 6.
Добавим поддержку @angular/material для нашего проекта.
```bash
ng add @angular/material @angular/cdk
```
Теперь создадим навигационную страницу нашего приложения
```bash
ng g @angular/material:material-nav --name=nav -m app
```
Настало время внести изменения в код нашего проекта в ручном режиме.
```html
// src/app/app.component.html
<app-nav></app-nav>
```
```diff
 // src/app/nav/nav.component.ts
 @Component({
-  selector: 'nav',
+  selector: 'app-nav',
   templateUrl: './nav.component.html',
   styleUrls: ['./nav.component.css']
 })
```



Создадим модуль
```bash
ng g m tables -m app --routing
```
В созданном модуле `tables` создадим компонент с уже готовой разметкой и стилями.
```bash
ng g @angular/material:material-nav --name=table
