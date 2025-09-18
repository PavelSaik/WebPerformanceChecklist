
Чек-лист оптимизации скорости загрузки страниц сайта
====================
Инструменты анализа скорости загрузки сайта с выдачей рекомендаций:

* [GTmetrix](http://gtmetrix.com/)
* [Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)
* [Web Vitals Extension](https://chrome.google.com/webstore/detail/web-vitals/ahfhijdlegdabablpippeagghigmibma)
* [WebPageTest](https://www.webpagetest.org/)
* Lighthouse CI для автоматизированного тестирования

GTmetrix использует Google Page Speed + Yahoo! YSlow и выдает подробные рекомендации, зато Google PageSpeed Insights
проверяет также загрузку на мобильных устройствах.

_Сравнивая GTMetrix и Google PageSpeed Insights (GPSI), следует помнить, что они используют разные алгоритмы и метрики для оценки производительности веб-сайтов. Важно учитывать оба
инструмента, но не полагаться исключительно на один из них. 🧐_

## Серверная оптимизация

1. Включение HTTP2 (обязательно нужен SSL).
2. Включение HTTP/3 (QUIC) &mdash; следующее поколение после HTTP/2 с улучшенной производительностью. Для РФ не применимо. 
2. [Включение gzip](http://gtmetrix.com/enable-gzip-compression.html) (рекомендуем значение 6) или включение современного сжатия: Brotli (приоритет).
3. Включение кеширования статики nginx (365d или max).
3. Включение кеширования генерации страниц движком сайта (в file, reddis или memcached).
4. Включение кеширования для файлов, отдаваемых сервером:
    1. [Last-Modifed](http://last-modified.com/ru/)
    2. [Expires headers](http://gtmetrix.com/add-expires-headers.html)
    2. [E-tag](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#validating-cached-responses-with-etags)
    3. [Cache-Control](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#cache-control)
    4. [Vary: Accept-Encoding header](https://www.maxcdn.com/blog/accept-encoding-its-vary-important/)
5. Проверка чтоб не было 404-тых откликов на загрузку ресурсов — они замедляют загрузку.
6. Проверка чтоб не было заблокированных ресурсов РКН, например пиксель от Facebook — они замедляют загрузку.

## Оптимизация скорости загрузки страниц

1. Предварительная [подготовка соединений](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/preconnect) и запросов для ресурсов, что грузятся со сторонних сервисов и CDN: `<link rel="preconnect" href="https://example.com">`, но не более 5. **Важно**: определить приоритетные ресурсы для загрузки.
2. Добавление спекулятивных роутов: 
```html
<script type="speculationrules"> 
{ "prerender": [ { "where": { "and": [ { "href_matches": "/*" }, { "not": { "selector_matches": ".no-prerender" } }, { "not" : {"selector_matches" : "[target=_blank]"}}, { "not": { "selector_matches": "[rel~=nofollow]" } } ] }, "eagerness" : "moderate" } ]}
</script>
```
3. Подключение CSS и JS &mdash; в конце HTML, перед `</body>`, или использовать defer/async из `head`, никакой разницы тогда не будет
4. Автоматически генерировать [критический CSS](https://github.com/addyosmani/critical) и вставлять его через `<style>`
   в `<head>`(использовать только для SPA приложений)
5. Минимизировать кол-во загружаемых файлов (не очень актуально, если есть HTTP2):
    1. Объединять все css в один файл
    2. Объединять все js в один файл, если файл больше 250кб использовать чанки
    3. Использовать только [WOFF2](http://caniuse.com/#search=woff2) при подключении web fonts. [Подробнее](http://bdadam.com/blog/better-webfont-loading-with-localstorage-and-woff2.html) про подключение. Конвертер шрифтов тут https://transfonter.org, стоит использовать только нужные глифы, например, только русский и английский язык.
6. Отложить загрузку данных необязательных для первого отображения страницы [Вынести кнопки соц. шаринга в пост-загрузку](https://github.com/ideus-team/bem-snippets/blob/master/js-socialSharePreload/README.md) или загружать их при прокрутке содержимого к ним
7.  **Всегда** использовать нативный LazyLoad для картинок и iframe. Прописать `loading="lazy"` для **img** и **iframe** и `decoding="async"` **img** для в html. 
8. Для изображений которые должны быть сразу видны пользователю при загрузке использовать `loading="eager"`
9. ⚠️ **Важно**: обязательно указывать размеры изображений (width/height) при использовании `decoding="async"` для предотвращения CLS (Cumulative Layout Shift).
10. Перенести внешние баннеры и другие ресурсы подгружаемые со сторонних медленных серверов — на сервер клиента.
11. Минимизировать редиректы для внешних ресурсов (например внешний js отдается не по тому URL, по которому
    запрашивается, а по редиректу стого URL).
12. Минимизировать CSS, JS и HTML.
13. Оптимизировать графику через https://squoosh.app/, использовать плагины для gulp/webpack.
	1. Использовать тег `picture` для подгрузки современных форматов: AVIF (приоритет), WebP, затем fallback.
    2. Устанавливать качество 85–90.
	3. Если изображение в макете как background, все равно использовать picture с тегами lazyload и object-fit: cover.
	4. Перенос визуальных украшений в CSS3 вместо картинок:
		- например, тень у картинки можно сделать через box-shadow, а саму картинку сохранить без тени;
		- **всегда** выгружать из макета изображения без скруглений.
14. Внести правки в дизайн, удалив тяжеловесные элементы.
15. Желательно удалить query string ("?") в URL отдаваемых ресурсов (некоторые прокси не кешируют их).
16. Оптимизация Core Web Vitals метрик:
    - LCP (Largest Contentful Paint) < 2.5s
    - FID (First Input Delay) < 100ms
    - CLS (Cumulative Layout Shift) < 0.1
    - INP (Interaction to Next Paint) < 200ms (заменяет FID)
17. Использовать ES6 модули с `type="module"` и `nomodule` для legacy браузеров.
18. Реализация Service Worker для кеширования ресурсов и offline-функциональности.

Этот чек-лист создан командой [студии Павла Сайка](https://palpalych.ru/ "Комплексные решения интернет-маркетинга") для оптимизации производительности веб-сайтов и максимальной оптимизация скорости загрузки страниц.
