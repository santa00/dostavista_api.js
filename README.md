# Javascript-клиент для API dostavista.ru

**Задача библиотеки:** облегчить интеграцию интернет-магазинов со службой доставки Dostavista.ru.

JS-клиент предназначен для подключения к админке интернет-магазина, работает поверх XMLHttpRequest и предоставляет простой способ отправлять заказы в службу доставки.

## Правила работы

1. **Скачать последнюю версию библиотеки из `build/` и подключить** к странице, с которой планируется отправлять заказы в Достависту.

	```html
	<head>
		...
		<link rel="stylesheet" href="dostavista_api.min.css">
	</head>

	...
	<script src="dostavista_api.min.js"></script>
	</body>
	```

2. **Вставить тег `<span class="DostavistaButton"></span>`** рядом с каждым заказом. Одна кнопка отправляет в Достависту один заказ.

3. **Добавить dsta-аттрибуты, описывающие каждый заказ.** Для каждого тега `.DostavistaButton` в разметке на момент инициализации Javascript должны содержаться все необходимые параметры, либо можно воспользоваться колбэком `onBeforeSend` (см. ниже).

	```html
	<span class="DostavistaButton"
		...
		dsta-matter="Видеорегистратор"
		dsta-insurance="15000"
		...

		></span>
	```

	**Полный список всех атрибутов**

	| Атрибут | Тип и ограничения | Значение |
	|---------|-------------------|----------|
	| `dsta-matter` | Строка | Что везём, например «диктофон» или «видеорегистратор» |
	| `dsta-insurance` | Целое число от 0 до 15000 | Сумма страховки в рублях |
	| `dsta-address` | Строка | Адрес первой точки маршрута |
	| `dsta-required_time_start` | YYYY-MM-DD HH:MM:SS | Время прибытия на точку (от) * обязательно |
	| `dsta-required_time` | YYYY-MM-DD HH:MM:SS | Время прибытия на точку (до) * обязательно |
	| `dsta-contact_person` | Строка | Имя контактного лица на точке забора |
	| `dsta-phone` | 10 цифр без знака + и международного кода | Телефон контактного лица на точке забора |
	| `dsta-weight` | Целое число | Вес на точке | 
	| `dsta-taking` | Целое число | Стоимость на точке: сумма, которую должен взять курьер на точке |
	| `dsta-client_order_id` | Строка | Номер заказа в магазине, используется для уведомлений на точках |


	> Копейки и десятые доли килограмма отбрасываются, даже если их указать.


4. **Задать общие для всех обращений к API Достависты параметры: `client_id` и `token`** для правильной авторизации в API. Метод `setClient` не делает AJAX-запросов, а просто сохраняет общие параметры на будущее.

	```html
	<script>
		DostavistaApi.setClient({
			client_id: 1234,
			token: 'xxxx'
		});
	</script>
	```

5. **Задать общий адрес**, если все заказы доставляются из одной точки «А» по разным адресам.

	```html
	<script>
		DostavistaApi.setDefaultFrom({
			client_order_id: ""
			weight: ""
			phone: ""
			contact_person: ""
			required_time: ""
			required_time_start: ""
			address: ""
	</script>
	```

6. **Задать необходимые колбэки**. Доступные варианты перечислены в таблице ниже.

	```html
	<script>
		DostavistaApi.setCallback('onSendSuccess', function(result) {
			alert('Заказ ' + result.order_id + ' доставлен');
		});
	</script>
	```

	**Доступные колбэки**

	| Название | Параметры | Описание |
	|----------|-----------|----------|
	| onBeforeSend |  | Вызывается до отправки данных и парсинга параметров. Можно использовать, чтобы добавить отсутствующие параметры. Должен возвращать `jQuery.Deferred.promise()` — см. код примера `api_test.html`! |
	| onSendSuccess | result | Выполняется, если заказ отправлен в Достависту. Получает result — объект с ответом API. |
	| onSendError | jqxhr, text, error | Выполняется, если при отправке произошла ошибка. Получает параметры, идентичные методу $.ajax().fail(). |


## Вопросы

1. В каком часовом поясе указывается время? Не будет ли ошибок с конвертацией? Можно попробовать перейти на [ISO-8061](http://ru.wikipedia.org/wiki/ISO_8601) с обязательным указанием часового пояса.


## TODO 
1. Отправка множества точек одним вызовом
2. Сделать атрибуты для задания своей точки забора для каждого заказа.
3. Заменить URL на живой в релизе.


### Технический TODO
1. Отказаться от jQuery
2. Тесты
3. Запихнуть весь код в [песочницу](https://github.com/a-ignatov-parc/requirejs-sandbox)

### Текущий TODO
1. Продумать как следует юзкейсы
2. Обновить README

3. Класть стили текстом в <style> из скрипта
4. Как это сделать с grunt.js?


4. Сделать методы для установки первого адреса по-умолчанию.
5. Заменить дата-атрибуты на JSON на `dsta-from="{}"` и `dsta-to="{}"`.

6. Обработка ошибок API!

7. Найти ошибку с неправильным типом point и рассказать о ней Игорю

12. Добавить возможность устанавливать несколько колбеков на событие

13. Доработать API и клиент, чтобы можно было менять кнопки «В Достависту» на номер заказа со ссылкой.

14. Нормальная обработка JSONP-ошибок.
15. Переделать логику состояний кнопки

16. Оформить как bower-модуль