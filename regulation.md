# Для чего нужно следовать этому регламенту

Вот основные цели:

1.  Облегчить жизнь разработчикам. Когда доработки выполняются в едином стиле, то разработчики тратят меньше времени и сил на чтение/разбор кода (и своего, и чужого).
2.  Минимизировать ошибки и недочеты. Есть нюансы, о которых разработчики часто забывают. Регламент о них напоминает.
3.  Упростить обновления конфигурации. Регламент описывает, как минимизировать изменение типовых объектов конфигурации.
4.  Облегчить работу пользователей. Чтобы интерфейс выглядел привычным, интуитивно понятным и легко осваивался, необходима его унификация по стандартам 1С.

# Объекты метаданных и роли

## Создание новых объектов конфигурации

При проектировании доработок рекомендуется преимущественно создавать новые объекты конфигурации, не изменять типовые.

При добавлении нового объекта конфигурации:

*   Имена новых объектов начинаем с префикса «нп\_». В синониме префикс не нужен.
*   Имена реквизитов и значения перечислений, добавляемые к **типовым объектам** также начинаем с префикса «нп\_». Исключение – стандартные реквизиты (например, Организация, Подразделение), которые могут использоваться в типовых механизмах.
*   Префикс в имени не нужен, если добавляем подчиненный объект (реквизит, форму и т.д.) к ранее добавленному и уже имеющему префикс.
*   Размещаем добавленные объекты после типовых, сортируем в алфавитном порядке. Добавляем в подсистему “нп\_ДобавленныеОбъекты”.
*   Добавляем атомарные роли на новый объект ([см. подробнее про роли](#Добавление-новых-ролей)):
*   Не забываем выносить новые объекты в интерфейс (если это необходимо).
*   Новые объекты стараемся добавлять в основную конфигурацию (не в расширение). [См. подробнее про расширения](#Использование-расширений).

## Изменение типовых объектов конфигурации

Рекомендуется минимизировать внесение изменений в типовые объекты. Удаление типовых объектов не допускается.

При изменении типового объекта конфигурации:

*   В настройках поддержки включаем возможность изменения только для изменяемого объекта (не снимаем замок с подчиненных или всей конфигурации).
*   Имена добавляемых реквизитов, значений перечислений, предопределенных значений начинаем с префикса «нп\_». Исключение – стандартные реквизиты (например, Организация, Подразделение), которые могут использоваться в типовых механизмах.
*   Изменяемые типовые объекты добавляем в подсистему «нп\_ИзмененныеОбъекты».
*   Для изменения логики работы типовых объектов рекомендуется использовать механизм [подписок на события](#подписки-на-события) (в частности для событий записи, заполнения, проведения). Если подпиской на событие решать задачу не целесообразно, то рассматриваем варианты [изменения типовых модулей](#изменение-типовых-модулей).

## Добавление новых ролей

Новые роли создаем атомарными

*   Для ссылочных объектов (справочники, документы и т.д.), а также независимых регистров сведений добавляем роли:
    *   на добавление / изменение: «нп\_ДобавлениеИзменение<ИмяОбъектаБезПрефикса\>»
    *   на чтение: «нп\_Чтение<ИмяОбъектаБезПрефикса\>»
*   Для отчетов – на использование: «нп\_Просмотр<ИмяОтчетаБезПрефикса\>»
*   Для обработок – на использование: «нп\_Использование<ИмяОтчетаБезПрефикса\>»
*   Для подсистем-разделов интерфейса роль «нп\_Раздел<ИмяПодсистемыБезПрефикса\>»
*   Для подчиненных регистров (с регистратором) роль на чтение: «нп\_Чтение<ИмяРегистраБезПрефикса\>»

При добавлении новой роли

*   Проверяем, что в настройках роли сняты флаги
    *   «Устанавливать права для новых объектов»,
    *   «Независимые права подчиненных объектов»
*   Проверяем, что для объекта сняты права
    *   Удаление
    *   Все варианты интерактивного удаления
*   Проверяем, что сняты все права на уровне конфигурации (Веб клиент, Мобильный клиент и т.д.)
*   Настраиваем RLS (если нужно). Шаблоны ограничений и настройки ограничений рекомендуется копировать из типовых ролей и адаптировать под свои объекты.
*   Добавляем роль в подсистему “нп\_ДобавленныеОбъекты”.

Подсистема “Управление доступом” на ИТС: [https://its.1c.ru/db/bsp313doc#content:73:1](https://its.1c.ru/db/bsp313doc#content:73:1).

***Типовые роли изменять не рекомендуется. При необходимости можно скопировать типовую роль и переработать под новые требования.***

## Подписки на события

Рекомендуется создавать подписки следующим образом:

*   На каждую пару «вид объекта метаданных» / «событие» создаем только одну подписку. Например, «нп\_ДокументыОбработкаПроведения» или «нп\_СправочникиПередЗаписью»
*   Источником указываем только те объекты, события которых планируем обрабатывать.
*   Обработчик подписки размещаем в серверном общем модуле «нп\_ОбработчикиСобытий».
*   В этом обработчике **не реализуем логику**, а только выполняем маршрутизацию – вызов конечной процедуры-обработчика (в которой уже реализована логика обработки) в зависимости от типа источника события.
*   Конечные процедуры-обработчики размещаем в других модулях.

# Интерфейс

*   Следуем рекомендациям по проектированию интерфейсов от 1С: [https://its.1c.ru/db/v8std/browse/13/-1/7](https://its.1c.ru/db/v8std/browse/13/-1/7)
*   Особенно обращаем внимание на рекомендации по компоновке форм: [https://its.1c.ru/db/v8std#content:722:hdoc](https://its.1c.ru/db/v8std#content:722:hdoc)
*   Следуем рекомендациям использования конкретных видов элементов на форме: [https://github.com/Oxotka/1CDesignGuide](https://github.com/Oxotka/1CDesignGuide)

## Изменение командного интерфейса (разделов)

*   Все добавляемые объекты метаданных (справочники, документы) за редким исключением выносим в какой-нибудь раздел командного интерфейса.
*   В типовые подсистемы свои объекты не добавляем. Вместо этого создаем подчиненные подсистемы с префиксом «нп\_», и уже в эти подсистемы добавляем разрабатываемые объекты.
*   Если нужно добавить новую команду в командный интерфейс существующей типовой подсистемы – заимствуем ее в расширение, и в контексте расширения добавляем новую команду в подсистему.

## Изменение типовых форм

Рекомендуется изменять типовые формы программно, в расширении:

1.  Заимствуем форму в расширение.
2.  Добавляем обработчик ПриСозданииНаСервере, в котором реализуем добавление / изменение элементов и реквизитов формы программно (можно вынести этот код в отдельную процедуру – «ИзменитьФорму», например). Примеры кода можно взять здесь: [https://infostart.ru/1c/articles/1118319/](https://infostart.ru/1c/articles/1118319/)
3.  Новые обработчики событий элементов формы размещаем также в модуле заимствованной формы.

Если требуются значительные изменения формы, которые сложно внести программно, то предлагается рассмотреть варианты:

1.  Разработка новой формы, содержащей объемный функционал, и возможность открытия ее из типовой.
2.  Прямое редактирование формы в расширении, и добавление подробного комментария в начале модуля формы с перечнем доработок.
3.  Разработка новой формы и подмена ею типовой (настоятельно **не рекомендуется** копирование типовой формы с последующей доработкой!).

## Добавление новых форм

*   Формы, добавляемые к типовым объектам именуем с префиксом «нп\_».
*   При разработке форм списка / выбора добавляем поле «Ссылка» с включенной настройкой «Видимость», но отключенной «Пользовательской видимостью».
*   Запросы для динамических списков стараемся разрабатывать максимально производительными.

## Изменение макетов

*   Прямое изменение типовых макетов в основной конфигурации запрещено.
*   Рекомендуется использовать механизм внешних печатных форм: вытаскиваем функционал типовой печатной формы во внешнюю обработку или расширение (или находим подходящую на Инфостарте), и вносим в нее необходимые изменения.
*   Небольшие изменения в некоторые типовые печатные формы можно внести через механизм настройки макетов печатных форм (в режиме Предприятия).
*   Если варианты выше не подходят, то рассмотреть такие:
    *   Заимствуем макет в расширение и дорабатываем в расширении.
    *   Копируем типовой макет (имя нового макета – «нп\_<ИмяТиповогоМакета>», добавляем в подсистему «нп\_ДобавленныеОбъекты») и дорабатываем его. Подменяем в коде макет на новый  (см. [изменение типовых модулей](#изменение-типовых-модулей)).

# Кодирование

## Изменение типовых модулей

Описанные ниже правила относятся ко всем модулям (общим, объекта, менеджера, формы).

*   Типовые модули допускается дорабатывать только в том случае, если другим способом решать задачу нецелесообразно или невозможно.
*   Способы изменения типовых методов (в порядке предпочтения):
    1.  Заимствуем в расширение, если это допустимо (см. [раздел про использование расширений](#Использование-расширений)).
    2.  Переопределяем или дополняем функциональность метода в основной конфигурации:
        1.  Создаем новый модуль «нп\_<ИмяОригинальногоМодуля>» с настройками контекста, аналогичными исходному модулю.
        2.  Если метод в модуле документа, то добавляем общий модуль «нп\_Документ<ИмяДокумента>» (для справочников и других видов метаданных – по аналогии).
        3.  В новом модуле создаем экспортный метод с таким же именем, как и изменяемый. Реализуем в нем необходимый функционал.
        4.  В исходном модуле выполняем вызов нового метода, размещая его
            *   в конце исходного метода, если нужно дополнить его функциональность
            *   в начале исходного метода, после чего выполняется “Возврат”, если нужно заменить функциональность.
    3.  Дорабатываем метод в типовом модуле напрямую.
*   Новые методы создаем только в добавленных или заимствованных в расширение модулях. В типовых модулях новые методы не создаем.

## Добавление общих модулей

*   Имя модуля должно быть с префиксом «нп\_», модуль включаем в подсистему «нп\_ДобавленныеОбъекты»
*   Руководствуемся правилами создания общих модулей [https://its.1c.ru/db/v8std#content:469:hdoc](https://its.1c.ru/db/v8std#content:469:hdoc)
*   Новые общие модули добавляем в основной конфигурации. В расширении их создаем исключительно в тех случаях, когда функционал модуля предназначен для использования строго в рамках этого расширения.

## Маркировка программного кода

*   Изменяемый и добавляемый код отмечаем комментариями-маркерами, которые содержат следующие данные:
    *   исполнитель,
    *   основании изменения (номер заявки или название задачи),
    *   дата внесения,
    *   признак начала и окончания изменений.
*   При необходимости заменить или удалить код – его не удаляем, а комментируем. **Но в соблюдении этого правила не нужно фанатизма, в целом код должен оставаться читаемым:** большие удаляемые блоки или “наслоения” комментариев при множественных изменениях одного блока кода лучше действительно удалять, отметив в комментарии, что код был удален.
*   При замене или удалении кода указываем причину изменений, какую проблему решает данное изменение.
*   Пример

```
// Харин 11.07.2023 Номер или название задачи {
// Причина изменений
// [...Удаляемый код...]
[...Добавляемый код...]
// Харин }
```

*   Комментарии-маркеры размещаем только внутри методов. Обрамлять ими методы «снаружи» не следует.
*   Если в разработке используем Git, то можно маркировать только изменения типового кода.

## Рекомендации по разработке программного кода

В коде на встроенном языке:

*   Именуем методы по правилам [https://its.1c.ru/db/v8std#content:647:hdoc](https://its.1c.ru/db/v8std#content:647:hdoc). У метода хорошее имя, если при чтении кода вам не нужно переходить к реализации метода (нажимать F12), чтобы понять что он делает.
*   Именуем переменные по правилам [https://its.1c.ru/db/v8std#content:454:hdoc](https://its.1c.ru/db/v8std#content:454:hdoc). У переменной хорошее имя, если для понимания ее смысла не нужно изучать как она используется.
*   Используем функциональность БСП. См. [https://infostart.ru/1c/articles/1398340/](https://infostart.ru/1c/articles/1398340/), [https://infostart.ru/1c/articles/1411756/](https://infostart.ru/1c/articles/1411756/)
*   Описываем экспортные методы по стандарту [https://its.1c.ru/db/v8std#content:453:hdoc](https://its.1c.ru/db/v8std#content:453:hdoc).
*   Размещаем методы в модуле по правилам [https://its.1c.ru/db/v8std#content:455:hdoc](https://its.1c.ru/db/v8std#content:455:hdoc).
*   Форматируем код так, чтобы чтение его не вызывало проблем. Если лень делать это вручную – есть Alt+Shift+F.
*   Избегаем длинных строк кода, которые не умещаются в ширину экрана. Прокрутка вправо очень усложняет чтение кода.
*   Формируем тексты сообщений с помощью СтрШаблон() вместо конкатенации строк.
*   Если есть вероятность, что в конфигурации будут работать не русскоязычные пользователи, то используем НСтр() для текстов интерфейса и сообщений пользователю.
*   В операциях деления всегда проверяем знаменатель на 0.
*   Проверяем признак ОбменДанными.Загрузка в обработчиках событий (см. [https://its.1c.ru/db/v8std#content:773:hdoc](https://its.1c.ru/db/v8std#content:773:hdoc)).
*   Используем модули повторного использования для кэширования и оптимизации (см. [https://its.1c.ru/db/v8std/content/724/hdoc](https://its.1c.ru/db/v8std/content/724/hdoc)).
*   В целом стараемся следовать соглашениям по написанию кода из системы стандартов [https://its.1c.ru/db/v8std#browse:13:-1:31](https://its.1c.ru/db/v8std#browse:13:-1:31), если они не противоречат правилам регламента.

В запросах:

*   Не забываем указывать РАЗРЕШЕННЫЕ. Или выполнять запрос в блоке Попытка, чтобы корректно обработать ограничение RLS (последнее делают редко).
*   Предпочитаем временные таблицы вложенным запросам. Запросы с ВТ легче читать.
*   В операциях деления всегда проверяем знаменатель на 0.

# Работа с хранилищем конфигурации

*   Захватываем только те объекты. которые нужны для выполнения задачи (например, не захватываем подчиненные формы документа, если нужно доработать модуль объекта).
*   Не держим захваченным корень конфигурации! При необходимости добавить новый объект в конфигурацию:
    *   Захватываем корень;
    *   Добавляем объекты;
    *   Отпускаем корень;
    *   Захватываем добавленные объекты.
*   Не помещаем в хранилище регистр, не указав его регистраторы. Помним о том, что после помещения объектов в хранилище, конфигурация там должна оставаться целостной.
*   Коммит в хранилище сопровождаем комментарием – номер или наименование задачи. Не будет лишним краткое описание изменений. Если коммит делается не от учетки автора, то указываем в комментарии также фамилию автора.

# Использование расширений

*   Создаем и изменяем объекты, связанных с таблицами базы данных, в основной конфигурации (если это возможно).
*   При принятии решения о реализации функционала в расширении учитываем, что в какой-то момент при запуске базы расширение может не подключиться. Соответственно в расширениях не рекомендуется реализовывать критически важную бизнес-логику.
*   Также учитываем необходимую частоту обновлений, важность актуализации типового функционала. Например, “1С: Бухгалтерия предприятия” я бы предпочел дорабатывать только расширениями.

# Тестирование функционала

*   Перед сдачей обязательно тестируем разработанный функционал. Даже если добавили всего одну строчку кода.
*   Выполняем тестирование
    *   под ограниченными правами.
    *   по сценарию работы пользователя, а не так как удобно программисту.

