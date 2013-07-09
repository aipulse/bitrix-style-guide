Стандарты программирования для платформы 1С-Битрикс
=====================
В этом руководстве описаны рекомендуемые техники при разработке под платформу [1С-Битрикс](http://www.1c-bitrix.ru/) (далее - Битрикс).

Весь PHP код должен написан в соответствии со стандартом [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md).

Стиль написания кода должен соответствовать стандарту [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md).

0. Общие положения
-----------

1. Оформление исходного кода
-----------
- Используйте кодировку `UTF-8` без `BOM` для файлов с исходным кодом.
- Используйте сокращенные теги `<?` и `<?=`.
 
    ```php
    // bad
     <?php echo .. ?>

    // good
    <?= .. ?>
    ```

- Используйте **ТАБ** для каждого отступа. Никаких пробелов для блоков кода.


2. Базовые правила при разработке под Битрикс
-----------
- При добавлении кода в файл `init.php` группируйте код в классы и выносите их в отдельные файлы.
 
    `init.php`
    ```php
    // SomeClass
    include_once($_SERVER["DOCUMENT_ROOT"] . '/bitrix/php_interface/includes/some_class.php');
    ```
- Не используйте **цифровые значения** в GetList, GetByID и схожих методах, которые принимают различные ID. Создайте файл со всеми необходимыми константами и используйте их имена. 
- У каждой константы должно быть **говорящее** имя и комментарий.

    ```php
    // bad
    $comments = CIBlockElement::GetList(Array(), Array("IBLOCK_ID" => 12));
    ```
    
	`// good`
	
    `constants.php` 
    
    ```php
    // ИБ с комментариями пользователей
    const COMMENTS_IBLOCK_ID = 12;
    ```
    Подключите этот файл в init.php
    ```php
    //Константы проекта
    include_once($_SERVER["DOCUMENT_ROOT"] . '/bitrix/php_interface/includes/constants.php');
    ```
    Используйте константу
    ```php
    $comments = CIBlockElement::GetList(Array(), Array("IBLOCK_ID" => COMMENTS_IBLOCK_ID));
    ```
- Используйте [Bitrix IBlock Tools](https://github.com/xescoder/bitrix-iblock-tools), [обсуждение](http://habrahabr.ru/post/185080/).
- При выборках данных (например, [GetList](http://dev.1c-bitrix.ru/api_help/iblock/classes/ciblockelement/getlist.php)) **обязательно** указывайте поля, которые нужны для дальнейших манипуляций, кроме случаев, когда нужны все поля:

    ```php
    // good - Обязательно указывайте поля для выборки
    $arSelect = Array("ID", "NAME", "DATE_ACTIVE_FROM");
    ...
    $res = CIBlockElement::GetList(Array(), $arFilter, false, Array(), $arSelect);
    ```
- При необходимости выбрать несколько элементов по ID, **обязательно** используйте GetList вместо GetByID:

    ```php
    // bad
    $element1 = CIBlockElement::GetByID(1);
    $element2 = CIBlockElement::GetByID(2);

    // good
    $elements = CIBlockElement::GetList(Array(), Array(1, 2));
    ```
- Не используйте **прямые запросы к базе** данных, вся работа только через [API](http://dev.1c-bitrix.ru/api_help/).
- Если к файлу предусмотрен прямой доступ через WEB, в первой строке файла добавьте:

    ```php
    <?if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die();?>
    ```


3. Работа с компонентами
-----------
- Шаблонам компонентов давайте осмысленные названия и в каждом проекте придерживайтесь общего стиля. Например, `Раздел/страница_сайта.Название.Тип`
Примеры:
 	- `index.user.auth`
	- `profile.orders.list`
 	- `cart.products.additional`
- Стандартные компоненты **не модифицируются**. Если возникнет такая необходимость — создайте копию компонента в своем пространстве имен в папке `/bitrix/components/`.
- РЕКОММЕНДУЕТСЯ все шаблоны компонентов сохранять в шаблоне `.default` в папке `/bitrix/templates/.default/`.
- НЕ РЕКОМЕНДУЕТСЯ делать любые манипуляции с данными в файле `template.php`. При необходимости правки логики стандартных компонентов, но недостаточной для того, что делать свой используются файлы [result_modifier.php](http://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=43&LESSON_ID=2830&LESSON_PATH=3913.4565.2830) и [component_epilog.php](http://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=43&LESSON_ID=2975&LESSON_PATH=3913.4565.2975)
- НЕЛЬЗЯ вставлять код вызова компонента внутрь файла `template.php` другого компонента.
- РЕКОМЕНДУЕТСЯ использовать файлы `style.css` и `script.js` в шаблонах только если они переопределяют стандартное поведение схожих элементов. РЕКОМЕНДУЕТСЯ оставить комментарий об этой особенности.
- Кэширование компонентов **не отключается**. Практически не существует задач, которые нельзя решить с включенным кэшированием.


4. Работа с шаблонами
-----------
- РЕКОМЕНДУЕТСЯ использовать минимальное количество шаблонов.
- РЕКОМЕНДУЕТСЯ подключать `header.php` и `footer.php` из одного места для всех шаблонов, если это позволяет дизайн и верстка.
- РЕКОМЕНДУЕТСЯ общие картинки, скрипты и стили шаблонов сохранять в одном месте, например, в `/bitrix/templates/.default/`.


5. Работа с инфоблоками
-----------
- Все свойства инфоблоков называть в верхнем регистре, осмысленно (используя связку сущность-наименование), разделяя слова [подчеркиванием](http://ru.wikipedia.org/wiki/%D0%9F%D0%BE%D0%B4%D1%87%D1%91%D1%80%D0%BA%D0%B8%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5). Например:
	- Имя пользователя: USER_NAME
	- Валюта заказа: ORDER_CURRENCY
	- Список заказов: ORDER_LIST