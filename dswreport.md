# DSW Reports — генератор отчетов DeepSeeWeb

Продукты [InterSystems Data Platform](https://www.intersystems.com/resources/detail/intersystems-iris-data-platform/) включают в себя аналитический модуль [DeepSee](https://www.intersystems.com/products/intersystems-iris/analytics/), предоставляющий возможность создавать интерактивные дешборды. Дешборды можно симпатично визулизировать при помощи приложения [DeepSeeWeb](https://github.com/intersystems-ru/DeepSeeWeb) и предоставить доступ пользователям, например [аналитика](https://analytics.community.intersystems.com/) [InterSystems Developer Community](https://community.intersystems.com/). Но иногда при решении бизнес задач разработчик сталкивается с необходимостью формирования отчётов из произвольных виджетов, расположенных в отдельных дешбордах. Затем печатать эти отчёты в PDF и отправлять по электронной почте, желательно по расписанию. Так и появился проект DSW Reports, который расширяет функционал DeepSeeWeb, и решает задачи оговоренные выше. В этой статье будет описано как пользоваться DSW Reports.
<cut />
## Что такое DSW Reports?
[DSW Reports](https://github.com/intersystems-community/dsw-reports) - это расширение DSW, написанное на AngularJS, которое реализует оновной функционал для автоматической генерации отчётов. DSW Reports использующет [DeepSeeWeb](https://github.com/intersystems-ru/DeepSeeWeb) для отрисовки виджетов и [MDX2JSON](https://github.com/intersystems-ru/Cache-MDX2JSON) для обработки MDX запросов.

### Возможности:
- Отрисовка выбранных виджетов с установленными фильтрами.
- Вывод результатов вычисления произвольных MDX запросов.
- Автоматическая печатать PDF отчётов и рассылка их по почте
- Кастомизация внешнего вида отчёта при помощи CSS стилей

![HTML report](https://raw.githubusercontent.com/MakarovS96/images/master/Report.png)

## Создание отчёта
Для формирования отчёта в DSW Reports достаточно создать как минимум 2 файла: 

- **index.html** - каркас и главная страница отчёта, обычно не изменяется.
- **config.js** - конфигурация  отчёта, меняется для разных отчётов, отвечает за наполнение отчёта.

В файле конфигурации очёта должна обязательно содержаться функция **getConfiguration**.
```javascript
// Общие настройки отчёта
function getConfiguration(params){...}
```
Функция **getConfiguration** принимает обьект *params*, который содержит в себе параметры из строки URL и дополнительный параметр "***server***", являющийся адресом сервера. Параметр "***server***" имеет вид: `protocol://host:port`.

Благодаря обьекту *params* можно передавать в отчёт любые данные через URL строку. Например, если требуется изменять фильтры виджетов по желанию, тогда передаём с URL параметр "***filter***" и он будет доступен через обьект *params*.
```javascript
//<protocol://host:port>/dsw/reports/report_dir/index.html?filter=NOW
function getConfiguration(params){
    var filter = params["filter"]; // filter = "NOW"
}
```

Функция **getConfiguration** возвращает обьект, содержащий 3 свойства:

- *REPORT_NAME* - название отчёта
- *BLOCKS* - массив блоков отчёта
- *NAMESPACE* - область с данными для отчёта

Рассмотрим подробнее массив блоков *BLOCKS*. Блок - обьект с настройками виджета, настройками вычисляемых полей и т.д.

### Вид блока:
```javascript
{
    "title": String,            //Заголовок блока
    "note": String,             //Замечания под блоком. Могут содержать HTML код
    "widget": {                 //Настройки iframe виджета:
        "url": String,          //URL источника для iframe
        "height": Number,       //Высота iframe
        "width": Number         //Ширина iframe
    },
    "totals":[{                 //Настройки значений вычисляемых с помощью MDX
        "mdx": String           //MDX запрос
        "strings": [{           //Строки значений из запроса
            "title": String,    //Заголовок строки. Может использовать HTML.
            "value": String,    //Значение строки по умолчанию
            "value_append": String, //Суффикс для значения. 
                                //Может использоваться для знаков %, $ и т.д. 
                                //% преобразует значение в процентное (x * 100).
                                //Может использовать HTML.
            "row": Number       //Номер строки MDX запроса, 
                                //из которой берётся значение. 
                                //По умолчанию 0.
        },{...}]
    },{...}]}
```
Все поля обязательны, если поле не нужно лучше выставить его пустой строкой.

<details>
<summary>Пример блока</summary>

```javascript
{
     title: "Persons",
     note: "",
     widget: {
        url: server + "/dsw/index.html#!/d/KHAB/Khabarovsk%20Map.dashboard" + 
        "?widget=1&height=420&ns=" + namespace,
        width: 700,
        height: 420
     }
}
```

</details>

<details>
<summary>Ещё пример</summary>

```javascript
{
    title: "Khabarovsky krai",
    note: "Something note (only static)",
    widget: {
        url: server + "/dsw/index.html#!/d/KHAB/Khabarovsk%20Map.dashboard" + 
        "?widget=0&height=420&isLegend=true&ns=" + namespace,
        width: 495,
        height: 420
    },
    totals: [{
       mdx: "SELECT NON EMPTY " + 
      "[Region].[H1].[Region].CurrentMember.Properties(\"Population\") ON 0,"+
      "NON EMPTY {[Region].[H1].[Region].&[Хабаровск]," + 
      "[Region].[H1].[Region].&[Комсомольск-на-Амуре],"+
      "[Region].[H1].[Region].&[Комсомольский район]} ON 1 FROM [KHABCUBE]",
       strings: [{
            title: "Khabarovsk: ",
            value: "None",
            value_append: " чел."
        }, {
            title: "Komsomolsk-on-Amur: <br />",
            value: "None",
            value_append: " чел.",
            row: 1
        }, {
            title: "Komsomolsky district: <br />",
            value: "None",
            value_append: " чел.",
            row: 2
        }]
    }]
}
```
</details>

### Чем заполнять блок?
Основные поля для заполнения в блоке - это **url** для настроек виджета и **mdx** для настроек вычисляемых значений.   
- **MDX** можно составить и вручную, но рекомендуется делать это при помощи визуального коструктора [Analyzer](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=D2ANLY_ch_intro), встроенного в DeepSee.
![Analyzer](https://raw.githubusercontent.com/MakarovS96/images/master/Analyzer.png)
- **URL** можно получить при помощи DeepSeeWeb. Виджеты встроенные в отчёт это элементы *iframe*, источниками которых являются виджеты DeepSeeWeb. Для того, чтобы получить ссылку на источник надо выбрать пункт *"Share"* в контекстном меню виджета.
![Share](https://raw.githubusercontent.com/MakarovS96/images/master/Share.png)

### Кастомизация внешнего вида отчёта. 
Вместе с библиотеками отчёта поставляется файл **style.css** позволяющий редактировать внешний вид отчёта. В нем содержится стандарный набор классаов управляющий всеми элементами отчёта. Также можно добавлять свои классы стилей и использовать их в файле **index.html**.

## Рассылка по E-mail

Допустим отчёт уже готов и размещён в папке отчетов в DeepSeeWeb. Т.е. интерактивный HTML отчет теперь доступен по ссылке. Что нужно сделать чтобы, конвертировать его в PDF и разослать по почте? Это автоматически сделают [pthantomjs](http://phantomjs.org/) и встроенный SMTP клиент. Как установить и настроить phantomjs можно посмотреть здесь ([windows](https://youtu.be/L8Lw53MjDdY), [ubuntu](https://www.vultr.com/docs/how-to-install-phantomjs-on-ubuntu-16-04)). Но для этого надо настроить SMTP клиент и создать задание в [Менеджере задач](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=GSA_manage_taskmgr). 

### Настройка SMTP
Все настройки производятся в терминале.
1. Сначала надо настроить почту для рассылки
```
// Функция для настройки SMTP
do ##class(DSW.Report.EmailSender).setConfig(server, port, username, 
                                                 password, sender, SSLConfig)
```
- **server** - адрес SMTP сервера.  
- **port** - порт для исходящих собщений.  
- **username** и **password** - аутентификационные данные.  
- **sender** - E-mail адрес рассылки.  
- **SSLConfig** - *Опционально*. Имя [SSL конфигурации](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=GCAS_ssltls).   
2. Затем следует настроить список пользователей для рассылки
```
// Функция для добавления пользователя
do ##class(DSW.Report.EmailSender).addRecipient(email)
// Функция для удаления пользователя
do ##class(DSW.Report.EmailSender).deleteRecipient(email)
```
3. После предыдущих шагов можно запустить рассылку
```
// Функция запускающая рассылку
do ##class(DSW.Report.Task).Run(url, reportname)
```
- **url** - ссылка на отчёт.  
- **reportname** - название отчёта. Используется при генерации PDF.

### Автоматический запуск рассылки
Для автоматизации рассылки воспользуемся [Менеджером задач](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=GSA_manage_taskmgr). Создадим новую задачу со следующими параметрами:
1. На первой странице настраивается область запуска и прописываем нашу функцию для запуска рассылки.
![Task1](https://raw.githubusercontent.com/MakarovS96/images/master/Task1.png)
2. На второй странице настраивается время и переодичность запуска задачи.
![Task2](https://raw.githubusercontent.com/MakarovS96/images/master/Task2.png)
3. Последним шагом надо нажать *"Завершить"*.

Всё, после всех этих манипуляций у нас получился автогенерируемый отчёт, состоящий из виджетов DeepSeeWeb, который в заданное время рассылается по почте в виде PDF.

Пример готового отчёта можно посмотреть [здесь](https://bit.ly/2MhMLfh).   
Файл конфигурации этого отчёта посмотреть [здесь](https://github.com/intersystems-community/dc-analytics/blob/master/src/reports/week/config.js).   
А [здесь](https://community.intersystems.com/post/analysing-developer-community-activity-using-intersystems-analytics-technology-deepsee) можно подписаться на еженедельную доставку отчетов.   
[Ссылка](https://github.com/intersystems-community/dsw-reports) на репозиторий.   