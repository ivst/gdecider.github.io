---
title: Оптимизация сайта для PageSpeed Insights | Статьи CMS "1С Битрикс"
keywords: битрикс PageSpeed, битрикс оптимизация, сжатие страниц битрикс
sidebar: articles_sidebar
folder: articles
permalink: articles-bx-site-optimisation-for-google-pagespeed-insights.html
summary: false
toc: false
---

### Задача

Провести оптимизацию выдачи контента Битрикс, контроль качества оптимизации провести при помощи PageSpeed Insights.

### Решение

* 1) Привести в соответствие изображения, таким образом, что бы браузеру не приходилось их масштабировать
* 2) Включить сжатие файлов в nginx
* 3) При помощи настроек битрикс перенести js и css в конец страницы
* 4) Минифицировать js и css при помощи gulp или иными средствами
* 5) Исключить служебные скрипты и стили битрикс из формирования пользовательских страниц сайта
* 6) Минифицировать html

#### Перенос js и css в конец страницы

##### JS 
Для корректной работы битрикс с файлами стилей и скриптов их нужно подключать при помощи функций Битрикс:

```php
<?
$asset = \Bitrix\Main\Page\Asset::getInstance();

// подключение стилей 
$asset->addCss(SITE_TEMPLATE_PATH."/some.css");

// подключение скриптов
$asset->addJs(SITE_TEMPLATE_PATH."/some.js");

// подключение произвольных кусков кода
$asset->addString('<link rel="canonical" href="https://' . $_SERVER['HTTP_HOST'] . $APPLICATION->GetCurPage(false) . '" />');
```

Для объединения всех стилей и скриптов в единый файл нужно в настройках Битрикс ```Настройки -> Настройки продукта -> Настройки модулей -> Главный модуль``` активировать галки **Объединять CSS файлы** и **Объединять JS файлы**

Так же можно проставить галки **Создавать сжатую копию объединенных CSS и JS файлов** и **Переместить весь Javascript в конец страницы**

Программно опцию "Переместить весь Javascript в конец страницы" можно активировать так:

```php
$asset->setJsToBody(true);
```
 
В случае программной активации галочка в админкеработать не будет, т.е. скрипты всегда будут помещаться в конец страницы.
 
Не все скрипты нужно помещать в футер. например tagManager от гугла должен быть вверху. Для исключения скрипта из переносимых нужно добавить атрибут **data-skip-moving=true** в тег script. Пример:
 
```js
<script data-skip-moving=true>
  // your script code
</script>
```

##### CSS

Для переноса CSS в конец страницы у Битрикса нет стандартных средств, но т.к. за вывод css отвечает отдельный метод, то мы можем вывести его в footer.php самостоятельно.

Для этого нужно вместо:

```php
<?
$APPLICATION->ShowHead();
?>
```

В файле **header.php** добавить:

```php
<?
// вместо $APPLICATION->ShowHead();
$bXhtmlStyle = true;
echo '<meta http-equiv="Content-Type" content="text/html; charset='.LANG_CHARSET.'"'.($bXhtmlStyle? ' /':'').'>'."\n";
$APPLICATION->ShowMeta("robots", false, $bXhtmlStyle);
$APPLICATION->ShowMeta("keywords", false, $bXhtmlStyle);
$APPLICATION->ShowMeta("description", false, $bXhtmlStyle);
$APPLICATION->ShowLink("canonical", null, $bXhtmlStyle);
$APPLICATION->ShowHeadStrings();
$APPLICATION->ShowHeadScripts();
?>
```

В файле **footer.php** добавить:

```php
<?
$APPLICATION->ShowCSS(true, $bXhtmlStyle);
?>
```