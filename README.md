enb-bem-i18n
============

[![NPM version](http://img.shields.io/npm/v/enb-bem-i18n.svg?style=flat)](https://www.npmjs.org/package/enb-bem-i18n) [![Build Status](http://img.shields.io/travis/enb-bem/enb-bem-i18n/master.svg?style=flat)](https://travis-ci.org/enb-bem/enb-bem-i18n) [![devDependency Status](http://img.shields.io/david/enb-bem/enb-bem-i18n.svg?style=flat)](https://david-dm.org/enb-bem/enb-bem-i18n)

Пакет предоставляет набор ENB-технологий для сборки файлов, обеспечивающих мультиязыковую поддержку проектов, созданных на основе БЭМ-методологии. Осуществляет сборку BEM.i18n, использующуюся в библиотеке [bem-bl](https://ru.bem.info/libs/bem-bl/dev/).

Основная задача пакета — собрать ключи и их значения для указанного языка и на основе полученных данных сформировать отдельные файлы с переводами.

Технологии:

* [i18n-merge-keysets](#i18n-merge-keysets) — служебная технология для сбора первичных данных.
* [i18n-lang-js](#i18n-lang-js) — технология, предоставляющая конечные файлы с сопоставленными переводами.
* [i18n-keysets-xml](#i18n-keysets-xml) — технология для локализации сервисов, использующих XSLT для шаблонизации.
* [i18n-bemjson-to-html](#i18n-bemjson-to-html)
* [bemhtml-i18n](#bemhtml-i18n) —
* [bh-bundle-i18n](#bh-bundle-i18n) – технология для сборки как клиентского, так и серверного BH-кода.

## Установка

```
npm install --save-dev enb-bem-i18n
```

**Требования к установке**

Зависимость от пакета `enb` версии `0.11.0` или выше.

## Принципы работы пакета

### Исходные данные

`keysets` — исходные данные для сборки файла с переводом. Представляют собой набор ключей и их значений (переводов строк).

Пример:

```javascript
module.exports = {
    scope: {
        key: "val"
    }
};
```

Ключ определяет, какое из значений должно быть выбрано для указанного языка. В качестве контекста (`scope`) обычно используется имя блока.

Такой набор данных создается отдельно для кажого языка хранится в файле `{lang}.js`. Одинаковые данные для всех языков, а также код ядра BEM.i18n  помещаются в файл `all.js`. Код ядра также хранится как набор данных (`keysets`) и подключается следующим образом: в `scope` используется значение `all`, в ключе (`key`) передается пустая строка.

**Расположение в файловой системе**

Файлы `{lang}.js`/`all.js` хранятся в папке `<block-name>.i18n` наряду с другими файлами технологий в папке блока.

```
common.blocks/
  ├──block1/
    ├──block1.js
    ├──block1.bh
    └──block1.i18n/
      ├──en.js
      ├──ru.js
      └──all.js

  ├──block2/
      ├──block2.js
      ├──block2.bh
      └──block2.i18n/
        ├──en.js
        └──ru.js
```

### Процесс сборки

Данные из файлов `{lang}.js` во время сборки проходят несколько этапов:

I. Файлы `{lang}.js` объединяются в один merge-файл (`?.keysets.{lang}.js`) с помощью технологии [i18n-merge-keysets](#i18n-merge-keysets). Набор языков, для которых будут собраны `?.keysets.{lang}.js`-файлы, задается в параметрах технологии.

Для блоков `block1` и `block2` результирующий `?.keysets.en.js`-файл будет собран следующим образом.

Исходный файл `en.js` блока `block1`:

```javascript
module.exports = {
    "block1": {
        "key1": "val1",
        "key2": "val2"
    }
};
```

Исходный файл `en.js` блока `block2`:

```javascript
module.exports = {
    "block2": {
        "key3": "val3"
    }
};
```

Результирующий `?.keysets.en.js`-файл:

```javascript
module.exports = {
    "block1": {
        "key1": "val1",
        "key2": "val2"
    },
    "block2": {
        "key3": "val3"
    }
};
```

II. `?.keysets.{lang}.js`-файл — это промежуточный результат сборки, который в дальнейшем используется технологией [i18n-lang-js](#i18n-lang-js).

Технология `i18n-lang-js` на вход принимает данные из `?.keysets.{lang}.js` и отдает функцию `BEM.I18N`, которая при вызове принимает ключ и отдает значение (строку) для конкретного языка. Результатом являются `lang.{lang}.js`-файлы, содержащие строки переводов, соответствующие ключам.


### Применение BEM.I18N

<a name="template-usage"></a>
#### Использование в шаблонах

Вызов `BEM.I18N` можно применять в шаблонах.

```javascript
block('block').content()(function() {
    return {
        elem: 'tooltip',
        content: BEM.I18N('block', 'tooltip');
    };
});
```

Для работы с разными языками используется один шаблон. Язык указывается в [опциях](#lang-js-options) сборщика и определяет, на основании каких [?.keysets.{lang}.js](#keysets-lang-js)-файлов создавать файл с переводом `lang.{lang}.js`.

<a name="js-usage"></a>
#### ? Использование в JavaScript

`block1/block1.i18n/ru.js`

```javascript
BEM.I18N('scope', 'key')
```


## Технологии

<a name="i18n-merge-keysets"></a>
### i18n-merge-keysets

Служебная технология. Собирает данные (`keysets`) для указанного языка в `?.keysets.{lang}.js`-файл на основе `{lang}.js`-файлов.

#### Опции

##### target

Тип: `String`. По умолчанию: `?.keysets.{lang}.js`.

Результирующий таргет.

`?.keysets.{lang}.js`-файл — это промежуточный результат сборки, который в дальнейшем используется технологией [i18n-lang-js](#i18n-lang-js).

<a name="core-bem-i18n"></a>
##### lang

Тип: `String`. Не имеет значения по умолчанию.

Язык, для которого небходимо собрать файл.

Допустимые значения:

* **'{lang}'** — указывает язык, для которого будут собраны данные (`keysets`) из файлов `{lang}.js`.
* **'all'** — подключает в результирующий файл `*.keysets.all.js` код ядра BEM.i18n из библиотеки `bem-bl`.

-------------------------------------
**Пример**

```javascript
nodeConfig.addTechs([
  [ require('enb-bem-i18n/techs/i18n-merge-keysets'), { lang: 'all' } ],
  [ require('enb-bem-i18n/techs/i18n-merge-keysets'), { lang: '{lang}' } ]
]);
```

### i18n-lang-js

Собирает `?.lang.{lang}.js`-файлы на основе данных из `?.keysets.{lang}.js`-файлов, полученных в результате использования [i18n-merge-keysets](#i18n-merge-keysets).

Технология `i18n-lang-js` генерирует модуль `BEM.I18N`, который после сборки `?.lang.{lang}.js`-файлов предоставляет возможность вызова функции `BEM.I18N` из [шаблонов](#template-usage) или [клиентского JavaScript](#js-usage).

Функция на вход принимает следующие параметры:

* **scope** — обычно имя блока.

* **key** — определяет, какое из значений должно быть выбрано для указанного языка.

Результат выполнения: строка, содержащая перевод.

#### Опции

##### target

Тип: `String`. По умолчанию: `?.lang.{lang}.js`.

Результирующий таргет.

**Пример**

```
if (typeof BEM !== 'undefined' && BEM.I18N) {BEM.I18N.decl('block1', {
    "key1": 'val1',
    "key2": 'val2',
}, {
"lang": "en"
});

BEM.I18N.decl('block2', {
    "key3": 'val3'
}, {
"lang": "en"
});

BEM.I18N.lang('en');

}
```

<a name="lang-js-options"></a>
##### lang

Тип: `String`. Не имеет значения по умолчанию.

Язык, для которого небходимо собрать финальный файл, содержащий строки переводов.

##### keysetsFile

Тип: `String`. По умолчанию — `?.keysets.{lang}.js`.

`?.keysets.{lang}.js`-файл — это результат выполнения [i18n-merge-keysets](#i18n-merge-keysets) — набор данных (`keysets`) для указанного языка, который используется технологией [i18n-lang-js](#i18n-lang-js) для формирования `?.lang.{lang}.js`-файлов.


**Пример**

```javascript
nodeConfig.addTechs([
  [ require('enb-bem-i18n/techs/i18n-lang-js'), { lang: 'all'} ],
  [ require('enb-bem-i18n/techs/i18n-lang-js'), { lang: '{lang}'} ]
]);
```
#### ??? Public API

### i18n-keysets-xml

Собирает `?.keysets.{lang}.xml`-файлы на основе `?.keysets.{lang}.js`-файлов.

Технология `i18n-keysets-xml` применяется для локализации сервисов, использующих [XSLT](https://github.com/veged/xjst) для шаблонизации.
Используется для локализации xml-страниц.

<a name="i18n-bemjson-to-html"></a>
### i18n-bemjson-to-html

Собирает *HTML*-файл с помощью *BEMJSON*, *BH* или *BEMHTML*, *lang.all* и *lang.{lang}*.

#### Опции

##### templateFile

Тип: `String`. Обязательный параметр.

Исходный файл шаблона.

##### bemjsonFile

Тип: `String`. По умолчанию — `?.bemjson.js`.

Исходный BEMJSON-файл.

##### langAllFile

Тип: `String`. По умолчанию — `?.lang.all.js`.

Исходный langAll-файл.


##### langFile

Тип: `String`. По умолчанию — `?.lang.{lang}.js`.

Исходный lang-файл.

Если параметр `lang` не указан, берется первый из объявленных в проекте языков.

##### target

Тип: `String`. По умолчанию — `?.{lang}.html`.

Результирующий HTML-файл.

------------------------------------
**Пример**

```javascript
nodeConfig.addTech(require('enb-bh/techs/i18n-bemjson-to-html'));
```

<a name="bemhtml-i18n"></a>
### bemhtml-i18n

Собирает `?.bemhtml.{lang}.js`-файлы на основе `?.keysets.{lang}.js`-файла и исходных шаблонов.

Склеивает *bemhtml.xjst* и *bemhtml*-файлы на основе заивисимостей (deps), обрабатывает `bem-xjst`-транслятором, сохраняет (по умолчанию) в виде `?.bemhtml.js`.

**Внимание:** поддерживает только JS-синтаксис.

#### Опции

##### target

Тип: `String`. По умолчанию — `?.bemhtml.js`.

Результирующий таргет.

##### lang

Тип: `String`. Не имеетзначения по умолчанию.

Язык, для которого небходимо собрать файл.


##### keysetsFile

Тип: `String`. По умолчанию — `?.keysets.{lang}.js`.

Исходный keysets-файл.

##### filesTarget

Тип: `String`. По умолчанию — `?.files`.

files-таргет, на основе которого получается список исходных файлов (его предоставляет технология `files`).

##### sourceSuffixes

Тип: `String`. По умолчанию — `['bemhtml', 'bemhtml.xjst']`.

Суффиксы файлов, по которым строится `files`-таргет.

##### exportName

Тип: `String`. По умолчанию — `'BEMHTML'`.

Имя переменной-обработчика BEMHTML.

##### devMode

Тип: `Boolean`. По умолчанию — `true`.

Development-режим.


##### cache

Тип: `Boolean`. По умолчанию — `false`.

Кэширование. Возможно только в production-режиме.

##### modulesDeps

Тип: `Object`.

Хэш-объект, прокидывающий в генерируемую для скомпилированных шаблонов обвязку, необходимые YModules-модули.


------------------------------------
**Пример**

```javascript
nodeConfig.addTech([ require('enb-bem-core-i18n/techs/bemhtml-i18n'), { lang: {lang}, devMode: false } ]);
```

<a name="bh-bundle-i18n"></a>
### bh-bundle-i18n

Собирает *BH*-файлы по зависимостям (deps) в виде `?.bh.js`-бандла на основе `?.keysets.{lang}.js`-файла.

Предназначен для сборки как клиентского, так и серверного BH-кода. Предполагается, что в BH-файлах не используется `require`.

Поддерживает CommonJS и [YModules](https://ru.bem.info/tools/bem/modules/). Если в исполняемой среде нет ни одной модульной системы, то модуль будет предоставлен в глобальную переменную `bh`.

#### Опции

##### target

Тип: `String`. По умолчанию — `?.bh.js`.

Результирующий таргет.


##### filesTarget

Тип: `String`.

files-таргет, на основе которого получается список исходных файлов.

##### lang

Тип: `String`.

Язык, для которого небходимо собрать файл.

##### keysetsFile

Тип: `String`. По умолчанию — `?.keysets.{lang}.js`. (его предоставляет технология `files`). По умолчанию — `?.files`.

Исходный keysets-файл.

##### sourceSuffixes

Тип: `String`. По умолчанию — ['bh.js'].

Суффиксы файлов, по которым строится `files`-таргет.

##### sourcemap

Тип: `Boolean`.

Строит карты кода.

##### mimic

Тип: `String|Array`

Имена переменных/модулей для экспорта.

##### jsAttrName

Тип: `String`. По умолчанию — `data-bem`.

Атрибут блока с параметрами инициализации.

##### jsAttrScheme

Тип: `String`. По умолчанию — `json`.

Cхема данных для параметров инициализации.

Форматы:

`js` — Получаем `return { ... }`.

`json` — JSON-формат. Получаем `{ ... }`.

##### jsCls

Тип: `String|Boolean`. По умолчанию - `i-bem`.

Имя `i-bem` CSS-класса.

Для того, чтобы класс не добавлялся, следует указать значение `false` или пустую строку.

##### escapeContent

Тип: `Boolean`. По умолчанию - `false`.

Экранирование содержимого.

--------------------------------
**Пример**
```javascript
nodeConfig.addTech(require('enb-bem-core-i18n/techs/bh-bundle-i18n'));
```

Лицензия
--------

© 2014 YANDEX LLC. Код лицензирован [Mozilla Public License 2.0](LICENSE.txt).
