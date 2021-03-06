# Обработка вложенных URL

По умолчанию `vue-loader` автоматически обрабатывает стили и файлы шаблонов с помощью [css-loader](https://github.com/webpack/css-loader) и компилятора шаблонов Vue. В процессе компиляции, все использованные URL, такие как `<img src="...">`, `background: url(...)` и CSS `@import` будут **обработаны как зависимости модуля**.

Например, `url(./image.png)` будет преобразовано в `require('./image.png')`, а затем

``` html
<img src="../image.png">
```

будет скомпилировано в:

``` js
createElement('img', { attrs: { src: require('../image.png') }})
```

### Правила преобразования

- Если URL является абсолютным путём (например, `/images/foo.png`), он будет оставлен как есть.

- Если URL начинается с `.`, он будет истолковываться как относительный запрос модуля и разрешаться на основе структуры каталогов вашей файловой системы.

- Если URL начинается с `~`, то всё что после него будет истолковываться как запрос модуля. Это означает, что вы можете ссылаться на ресурсы даже внутри node_modules:

  ``` html
  <img src="~/some-npm-package/foo.png">
  ```

- (13.7.0+) Если URL начинается с `@`, то также будет истолковываться как запрос модуля. Это полезно если в вашей конфигурации webpack есть псевдоним для `@`, который по умолчанию указывает на `/src` в любом проекте, созданном через `vue-cli`.

### Связанные загрузчики

Так как `.png` это не JavaScript-файл, вам необходимо настроить webpack использовать [file-loader](https://github.com/webpack/file-loader) или [url-loader](https://github.com/webpack/url-loader) для их обработки. Проект создаваемый с помощью `vue-cli` уже сделает это за вас.

### Почему

Преимущества подобного подхода:

1. `file-loader` позволяет определить куда нужно скопировать и поместить файл, а также как именовать его с добавлением в имя хэша для лучшего кеширования. Кроме того, это означает что **вы можете просто поместить изображения рядом с вашим `*.vue` файлами и использовать относительные пути, основанные на структуре каталогов, не беспокоясь об адресах при развёртывании**. При правильной конфигурации, webpack будет автоматически заменять пути к файлам в корректные URL в итоговой сборке.

2. `url-loader` позволяет вставлять файлы как base-64 ссылки если они меньше указанного размера. Это позволит уменьшить количество HTTP-запросов при использовании маленьких файлов. Если же файл больше указанного порога, то он автоматически подключится с помощью `file-loader`.
