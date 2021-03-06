# Bitrix Frontend Bundler

Представляю вам свой подход к сборке JS/CSS на стороне сервера, в рамках проекта на 1С-Битрикс.

При использовании JS будем придерживаться стандарта ES2017, а для более простой и гибкой работы со стилями будем использовать SASS.

Поехали!

## Введение

Что и как собираем?

**Основной JS:** _/local/source/js/**/*.js_ 

Это любые JS-файлы в указанной папке. На выходе получите 2 файла: собранный оригинал и его ES5-версия. Так что можно смело использовать все прелести ES2017, не парясь о работе кода в каком-нибудь IE11.

Подключайте файлы динамически через JS: сначала определите поддерживается ли ES2017, потом подключайте соответствующий билд-файл (см. раздел _Подключение JS/CSS на странице_).

Результат: _/local/build/main.js_ и _/local/build/main.es5.js_

**Основные стили:** _/local/source/scss/**/*.scss_

Любые SCSS-файлы в указанной папке, кроме файлов импорта.

Компилируется в обычный CSS. 

Результат: _/local/build/main.css_



**Вспомогательный JS:** _/local/source/vendor/js/**/*.js_

Любые JS-файлы в указанной папке.

Результат: _/local/build/vendor.js_



**Вспомогательный CSS:** _/local/source/vendor/css/**/*.css_

Любые CSS-файлы в указанной папке.

Результат: _/local/build/vendor.css_




## Установка

Менеджером npm-пакетов выступит Yarn, а собирать всё будет силами Gulp.js.
Пример установки под CentOS:

```bash
$ wget https://dl.yarnpkg.com/rpm/yarn.repo -O /etc/yum.repos.d/yarn.repo
$ curl --silent --location https://rpm.nodesource.com/setup_6.x | bash -
$ yum install -y yarn
$ yarn global add gulp
```

## Управление

Если ваша система готова к работе с Yarn и Gulp, тогда кладём следующие файлы в корень сайта:
* _gulpfile.js_
* _package.json_
* _.babelrc_

После этого, находясь в корне сайта, делаем:
```bash
$ yarn install
```

Запустить сборку можно так:
```bash
$ yarn build
```

Также предусмотрен watch-режим - сборка отдельных билдов при изменениях исходников + перезапуск Gulp при изменении _gulpfile.js_:
```bash
$ yarn serve
```

## Подключение JS/CSS на странице

Сейчас поговорим о подключении JS/CSS. Собранные стили (_vendor.css_ и _main.css_) я подключаю на уровне PHP:

```php
$assetManager = \Bitrix\Main\Page\Asset::getInstance();

$assetManager->addCss('/local/build/vendor.css');
$assetManager->addCss('/local/build/main.css');

$assetManager->addJs('/local/source/tools/JsLoader.js');
```

А основной JS советую подключать динамически, в зависимости от поддержки ES2017. Для этих целей я написал маленький модуль. Пример:

```html
<script type="text/javascript">

    function onVendorLoad() {
        if(JsLoader.es2017) {
            JsLoader.load('/local/build/main.js');
        } else {
            JsLoader.load('/local/build/main.es5.js');
        }
    }

    JsLoader.load('/local/build/vendor.js', onVendorLoad);
</script>
```
Реализация этого модуля также есть в репозитории.

## Режим разработки

По-умолчанию файлы билда будут минифицированы и их отладка будет весьма затруднительной. Данный подход применим для продакшна. Для девелопмент-версий проектов вам следует определить переменную среды _DEV_SERVER_ со значением 1.

Например:

```bash
$ export DEV_SERVER=1
```
В данном случае файлы не будут минифицироваться + добавятся .map-файлы для отладки.


## Итог
Вот и всё! На дев-серверах используем _yarn serve,_ а в продакшене запускаем _yarn build_ при деплое (см. _deploy.php_).
