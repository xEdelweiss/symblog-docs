[Часть 1] - Настройка Symfony2 и шаблонизация
=============================================

Обзор
-----

Эта часть охватывает первые шаги разработки сайтов на Symfony2. Мы скачаем и установим
`стандартную редакцию <http://symfony.com/doc/current/glossary.html#term-distribution>`_
(Standard Distribution) Symfony2, создадим пакет (bundle) и соберем основные части HTML
шаблона. К концу раздела у вас получится сконфигурированный сайт привязанный к локальному
домену, например, ``http://symblog.dev/``. Зададим основную структуру страниц и заполним
и заполним их какими-нибудь данными.

Ниже мы рассмотрим такие моменты:

    1. Установка Symfony2
    2. Настройка локального домена
    3. Пакеты
    4. Маршрутизация
    5. Контроллеры
    6. Шаблонизация с использованием Twig

Загрузка и установка
--------------------

Как уже было сказано выше, мы будем использовать стандартную редакцию Symfony2. Она
включает в себя библиотеки ядра и основные пакеты, необходимые для создания
сайтов. Вы можете `скачать <http://symfony.com/download>`_ её с сайта Symfony2. Поскольку
мне не хочется повторять отлично написанную документацию из `книги <http://symfony.com/doc/current/book/index.html>`_
по Symfony2, пожалуйста, обратитесь к разделу `Установка и настройка Symfony2 (англ.) <http://symfony.com/doc/current/book/installation.html>`_
за подробностями. Там рассказывается о том, какой пакет скачивать, как устанавливать
требуемые сторонние библиотеки и какие права выставлять на директории.

.. warning::

    Уделите особое внимание секции `Устанавливаем права доступа (англ.) <http://symfony.com/doc/current/book/installation.html#configuration-and-setup>`_,
    где объясняются разные способы установки прав на ``app/cache`` и ``app/logs``, чтобы
    пользователь веб-сервера и пользователь командной строки могли получить к ним доступ
    на записаь.

Настройка локального домена
---------------------------

Под этот учебный проект мы создадим локальный домен ``http://symblog.dev/``, но вы
можете придумать что-то другое. Инструкции будут приведены для `Apache <http://httpd.apache.org/>`_
и рассчитаны на то, что вы знакомы с запуском и настройкой Apache на своей машине.
Если вы хорошо разбираетесь в настройке локальных доменов или используете другой
веб-сервер, вроде `nginx <http://nginx.net/>`_ — пропускайте эту часть.

.. note::

    Следующие действия были выполнены в Linux дистрибутиве Fedora,
    поэтому пути и тому подобное могут отличаться в зависимости от
    вашей операционной системы.

Начнем с создания виртуального хоста в Apache. Найдите его конфигурационный файл и
добавьте следующие параметры, не забыв изменить указать ``DocumentRoot`` и ``Directory``.
Путь к конфигурационному файлу Apache и его название может отличаться от системы к
системе. В Fedora он находится по адресу ``/etc/httpd/conf/httpd.conf``. Редактировать
этот файл необходимо с привилегиями ``sudo``.

.. code-block:: text

    # /etc/httpd/conf/httpd.conf

    NameVirtualHost 127.0.0.1

    <VirtualHost 127.0.0.1>
      ServerName symblog.dev
      DocumentRoot "/var/www/html/symblog.dev/web"
      DirectoryIndex app.php
      <Directory "/var/www/html/symblog.dev/web">
        AllowOverride All
        Allow from All
      </Directory>
    </VirtualHost>

Теперь добавьте новый домен в конец файла с хостами  ``/etc/hosts``.
Опять же, необходимы привилегии ``sudo``.

.. code-block:: text

    # /etc/hosts
    127.0.0.1     symblog.dev

Наконец, не забудьте перезагрузить сервис Apache. Это применит изменения
конфигурации, которые мы сделали.

.. code-block:: bash

    $ sudo service httpd restart

.. tip::

    Если вам постоянно приходится создавать новые локальные домены, можете упростить
    эту задачу используя `Динамические виртуальные хосты <http://blog.dsyph3r.com/2010/11/apache-dynamic-virtual-hosts.html>`_.

К этому моменту, у вас должна начать работать страница ``http://symblog.dev/app_dev.php/``.

.. image:: /_static/images/part_1/welcome.jpg
    :align: center
    :alt: Страница приветствия Symfony2

Если вы впервые попали на страницу приветствия Symfony2, осмотритесь, походите по
демо-страницам. На каждой из этих страниц можно найти кусочки кода (snippet),
демонстрирующие как каждая страница работает изнутри.

.. note::

    Вы могли заметить панель инструментов внизу экрана приветствия. Она предоставляет
    разработчику бесценную информацию о состоянии приложения, которая включает в себя
    время выполнения, потребление памяти, запросы к БД, состояние авторизации и еще
    много чего полезного. По-умолчанию, эта панель видна только при работе из ``dev``
    окружения, т.к. в рабочей (production) среде она будет представлять собой большой
    угорзу безопасности раскрывая множество внутренних данных. Мы еще будем ссылаться
    к этой панели по мере изучения новых возможностей.

Настройка Symfony: веб-интерфейс
----------------------------------

Symfony2 предоставляет веб-интерфейс для конфигурирования разных составляющих сайта,
например, баз данных. Для нашего проекта, нам понадобится БД, поэтому давайте попробуем
её настроить.

Зайдите на ``http://symblog.dev/app_dev.php/`` и нажмите кнопку Configure (настройка).
Заполните необходимые данные (в этом учебнике будем использовать MySQL, хотя вы можете
выбрать инуюю СУБД). Дальше следует генерация CSRF токена. Обратите внимание, предупреждения
(notice) могут информаировать о том, что файл ``app/config/parameters.ini`` недоступен
для записи и вам надо будет вручную скопировать содержимое текстового поля с настройками
в этот файл: ``app/config/parameters.ini`` (заменив оригинальное содержимое).

Пакеты: Структурные элементы Symfony2
----------------------------------

Пакеты (bundles) являются базовыми структурными элементами приложения на Symfony2.
По сути, сам фреймворк Symfony2 является пакетом. Пакеты позволяют разделять
функциональность для обеспечения повторного использования кода. Они инкапсулируют
всё что необходимо для работы реализованной в них функциональности, включая контроллеры,
модели, шаблоны и различные ресурсы, вроде изображений и CSS. Мы создадим пакет для
нашего сайта в пространстве имён (namespace) Blogger. Если вы не знакомы с пространствами имён
в PHP — потратьте время на их изучение, т.к. в Symfony2 они используются повсеместно.
Подробности о том, как реализован автозагрузчик можно увидеть на странице
`Автозагрузчик Symfony2 (англ.) <http://symfony.com/doc/current/cookbook/tools/autoloader.html>`_.

.. tip::

    Хорошее понимание пространств имён может избавить вас от множества распространенных
    проблем, с которыми вы можете столкнуться, когда структура каталогов не совсем корректно
    отвечает структуре пространств имён.

Создаем пакет
~~~~~~~~~~~~~

Чтобы инкапсулировать функционал блога нам понадобится создать пакет Blog.
Он будет содержать все необходимые файлы, так что вы сможете подключить его в
любой другой проект на Symfony2. В Symfony2 встроен инструмент для работы из
командной строки, который поможет нам в выполнении многих задач. Одной из таких задач
является генерация пакета.

Чтобы запустить генератор пакетов, запустите следующую команду. Вам будет задано
несколько вопросов по конфигурации нового пакета. Используйте значения по-умолчанию.

.. code-block:: bash

    $ php app/console generate:bundle --namespace=Blogger/BlogBundle --format=yml

По завершению работы генератора, у нас будет готов базовый пакет. Нам надо будет
сделать несколько изменений.

.. tip::

    Использование генераторов предоставляемых Symfony2 не обязательно, они созданы для
    упрощения работы. Вы можете вручную воссоздать структуру пакета со всеми
    его директориями и файлами. Но всё-таки, несмотря на опциональность использования
    генераторов, они позволяют также выполнить необходимые операции для получения готового
    работающего пакета. Одной из таких операций является регистрация пакета.

Регистрация пакета
..................

Наш новый пакет ``BloggerBlogBundle`` был зарегистрирован в ядре (Kernel), находящемся
в ``app/AppKernel.php``. Symfony2 требует регистрации всех пакетов, которые будут
использоваться в приложении. Может вы уже обратили внимание на то, что некоторые пакты
зарегистрированы только для ``dev`` или ``test`` окружения. Загрузка этих пакетов для
рабочего (production) ``prod`` окружения будет создавать лишнюю нагрузку обеспечивая
работу функций, которые там не будут использоваться. Код ниже показывает, где был зарегистрирован
наш пакет ``BloggerBlogBundle``.

.. code-block:: php

    // app/AppKernel.php
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
            // ..
                new Blogger\BlogBundle\BloggerBlogBundle(),
            );
            // ..

            return $bundles;
        }

        // ..
    }

Маршрутизация
.............

Маршруты пакеты были импортированы в основной файл маршрутов приложения ``app/config/routing.yml``.

.. code-block:: yaml

    # app/config/routing.yml
    BloggerBlogBundle:
        resource: "@BloggerBlogBundle/Resources/config/routing.yml"
        prefix:   /

Параметр prefix (префикс) позволяет нам установить префикс для всех маршрутов пакета.
В нашем случае, мы оставим значение по-умолчанию ``/``.
Если вы захотите, чтобы все маршруты нашего блога начинались с ``/blogger`` — измените
параметр на ``prefix: /blogger``.

Структура по-умолчанию
......................

В директории ``src`` был воссоздана стандартная структура пакета. В самом верху
находится директория ``Blogger``, указывающая конкретно на пространство имён
``Blogger``, в котором мы создали пакет. Ниже у нас находится папка ``BlogBundle``,
в которой находится сам пакет. Мы рассмотрим её содержимое в процессе прохождения
учебника. Если вы знакомы с MVC фреймворками, то назначение некоторых подпапок
будет для вас понятно уже из названия.

Контроллер по-умолчанию
~~~~~~~~~~~~~~~~~~~~~~~

В прцоессе генерирования пакета, Symfony2 создала контроллер по-умолчанию. Мы можем
запустить его, перейдя по пути ``http://symblog.dev/app_dev.php/hello/symblog``. Там
вы увидите простенькую страничку приветствия. Попробуйте изменить ``symblog`` в URL'е
на своё имя. Посмотрим, как это реализовано.

Маршруты
........

Файлом маршрутизации ``BloggerBlogBundle`` является ``src/Blogger/BlogBundle/Resources/config/routing.yml``,
который содержит следующее правило.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /hello/{name}
        defaults: { _controller: BloggerBlogBundle:Default:index }

Маршрут состоит из шаблона и каких-то стандартных для него значений. Шаблон сверяется
с URL'ом, а значения по-умолчанию указывают какой из контроллеров должен сработать,
если правила маршрута были удовлетворены. В шаблоне ``/hello/{name}``, плейсхолдер (placeholder)
``{name}}`` соответствует любому значению, т.к. никаких требований для него не указано.
Также, в маршруте не указано требований относительно локали, формата и HTTP методов, поэтому
запросы GET, POST, PUT и т.д. будут удовлетворять шаблону.

Если маршрут полсностью соответствует требованиям, он будет выполнен контроллером, указанным
в параметре _controller. Этот параметр ссылается на логическое имя (Logical Name) контроллера,
что позволяет Symfony2 найти подходящий файл. Пример выше выполнит действие (action) ``index``
контроллера ``Default``, находящегося по адресу ``src/Blogger/BlogBundle/Controller/DefaultController.php``.

Контроллер
..........

Контроллер в нашем примере совсем простой. Класс ``DefaultController`` наследуется от ``Controller``,
в котором реализованы некоторые полезные методы, например, ``render``, используемый ниже. Поскольку
наш маршрут использует плейсхолдер, то его значение будет передано как аргумент ``$name``. В
действии ``index`` вызывается только отвечающий за вывод шаблона метод ``render`` с указанием файла
``index.html.twig`` в подпапке Default папки с шаблонами view нашего пакета ``BloggerBlogBundle``.
Путь к шаблону указан в формате ``пакет:контроллер:шаблон``. В нашем примере, это будет
``BloggerBlogBundle:Default:index.html.twig``, который указывает на файл, физически
расположенный по адресу ``src/Blogger/BlogBundle/Resources/views/Default/index.html.twig``.
Различные вариации формата применяется для указания на шаблоны, находящиеся в разных частях
нашего приложения. Об этом еще будет написано.

В массиве мы передаем переменную ``$name`` в наш шаблон.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/DefaultController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('BloggerBlogBundle:Default:index.html.twig', array('name' => $name));
        }
    }

Шаблон (отображение)
.......................

Как вы могли уже заметить, шаблон очень простой. Он выводит Hello и добавляет
значение аргумента переданного из контроллера.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Default/index.html.twig #}
    Hello {{ name }}!

Наводим порядок
~~~~~~~~~~~~~~~

Так как нам не нужны некоторые файлы созданные генератором, мы их удалим.

Начем с удаления контроллера ``src/Blogger/BlogBundle/Controller/DefaultController.php``
вместе с его его директорией, где хранятся его шаблоны ``src/Blogger/BlogBundle/Resources/views/Default/``.
Наконец, удалите маршруты описанные в ``src/Blogger/BlogBundle/Resources/config/routing.yml``

Шаблонизация
------------

У нас есть два стандартных решения для шаблонизации в Syfmony2;
`Twig <http://www.twig-project.org/>`_ и PHP. Конечно, вы можете не использовать
ни один из них, а подключить какой-то другой шаблонизатор. Что возможно благодаря
`Dependency Injection Container <http://symfony.com/doc/current/book/service_container.html>`_ Symfony2.
Мы же будем использовать Twig по нескольким причинам.

1. Twig быстрый - шаблоны Twig компилируются в PHP классы, обеспечивая совсем небольшую нагрузку.
2. Twig краткий - Twig позволяет выполнять многие операции, без лишней многословности, в отличии от PHP.
3. Twig поддерживает наследование шаблонов - это моё любимое. Шаблоны могут расширяться и переопределять
   другие шаблоны, позволяя дочерним шаблонам менять стандартные значения предоставленные родителями.
4. Twig безопасный - в Twig по-умолчанию включено экранирование вывода. Более того, там реализована
   песочница для импортированных шаблонов.
5. Twig расширяемый - Twig идёт с боольшим числом основных функций, которые вы можете ожидать от
   шалонизатора, но на случай, когда вам понадобится что-то  comes will a lot of common core functionality that
   you'd expected from a templating engine, but for those occasions where you need
   some extra специфичное, Twig может быть легко расширен.

Это только некоторые преимщуества Twig'а. Больше причин его использования вы можете найти на официальном
сайте `Twig <http://www.twig-project.org/>`_.

Структура разметки
~~~~~~~~~~~~~~~~~~

As Twig supports template inheritance, we are going to use the
`Three level inheritance <http://symfony.com/doc/current/book/templating.html#three-level-inheritance>`_
approach. This approach allows us to modify the view at 3 distinct levels within the
application, giving us plenty of room for customisations.

Main Template - Level 1
.......................

Lets start by creating our basic block level template for symblog. We need 2
files here, the template and the CSS. As Symfony2 supports
`HTML5 <http://diveintohtml5.org/>`_ we will also be using it.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html" charset="utf-8" />
            <title>{% block title %}symblog{% endblock %} - symblog</title>
            <!--[if lt IE 9]>
                <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
            <![endif]-->
            {% block stylesheets %}
                <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
                <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
                <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
            <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
        </head>
        <body>

            <section id="wrapper">
                <header id="header">
                    <div class="top">
                        {% block navigation %}
                            <nav>
                                <ul class="navigation">
                                    <li><a href="#">Home</a></li>
                                    <li><a href="#">About</a></li>
                                    <li><a href="#">Contact</a></li>
                                </ul>
                            </nav>
                        {% endblock %}
                    </div>

                    <hgroup>
                        <h2>{% block blog_title %}<a href="#">symblog</a>{% endblock %}</h2>
                        <h3>{% block blog_tagline %}<a href="#">creating a blog in Symfony2</a>{% endblock %}</h3>
                    </hgroup>
                </header>

                <section class="main-col">
                    {% block body %}{% endblock %}
                </section>
                <aside class="sidebar">
                    {% block sidebar %}{% endblock %}
                </aside>

                <div id="footer">
                    {% block footer %}
                        Symfony2 blog tutorial - created by <a href="https://github.com/dsyph3r">dsyph3r</a>
                    {% endblock %}
                </div>
            </section>

            {% block javascripts %}{% endblock %}
        </body>
    </html>

.. note::

    There are 3 external files pulled into the template, 1 JavaScript and 2 CSS.
    The JavaScript file fixes the lack of HTML5 support in IE browsers pre version
    9. The 2 CSS files import fonts from
    `Google Web font <http://www.google.com/webfonts>`_.

This template marks up the main structure of our blogging website. Most
of the template consists of HTML, with the odd Twig directive. Its these
Twig directives that we will examine now.

We will start by focusing on the document HEAD. Lets look at the title:

.. code-block:: html

    <title>{% block title %}symblog{% endblock %} - symblog</title>

The first thing you'll notice is the alien ``{%`` tag. Its not HTML, and its
definitely not PHP. This is one of the 3 Twig tags. This tag is the Twig
``Do something`` tag. It is used to execute statements such as control statements and
for defining block elements. A full list of
`control structures <http://www.twig-project.org/doc/templates.html#list-of-control-structures>`_
can be found in the Twig Documentation. The Twig block we have defined in the
title does 2 things; It sets the block identifier to title, and provides a
default output between the block and endblock directives. By defining a block we
can take advantage of Twig's inheritance model. For example, on a page to
display a blog post we would want the page title to reflect the title of the
blog. We can achieve this by extending the template and overriding the title block.

.. code-block:: html

    {% extends '::base.html.twig' %}

    {% block title %}The blog title goes here{% endblock %}

In the above example we have extended the applications base template that first
defined the title block. You'll notice the template format used with the
``extends`` directive is missing the ``Bundle`` and the ``Controller`` parts,
remember the template format is ``bundle:controller:template``. By excluding the
``Bundle`` and the ``Controller`` parts we are specifiying the use of the application
level templates defined at ``app/Resources/views/``.

Next we have defined another title block and put in some
content, in this case the blog title. As the parent template already
contains a title block, it is overridden by our new one. The title would now
output as 'The blog title goes here - symblog'. This functionality provided by
Twig will be used extensively when creating templates.

In the stylesheets block we are introduced to the next Twig tag, the ``{{`` tag,
or the ``Say something`` tag.

.. code-block:: html

    <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />

This tag is used to print the value of variable or expression. In the above example
it prints out the return value of the ``asset`` function, which provides us with
a portable way to link to the application assets, such as CSS, JavaScript, and images.

The ``{{`` tag can also be combined with filters to manipulate the output before
printing.

.. code-block:: html

    {{ blog.created|date("d-m-Y") }}

For a full list of filters check the
`Twig Documentation <http://www.twig-project.org/doc/templates.html#list-of-built-in-filters>`_.

The last Twig tag, which we have not seen in the templates is the comment tag ``{#``.
Its usage is as follows:

.. code-block:: html

    {# The quick brown fox jumps over the lazy dog #}

No other concepts are introduced in this template. It provides the main
layout ready for us to customise it as we need.

Next lets add some styles. Create a stylesheet at ``web/css/screen.css`` and add
the following content. This will add styles for the main template.

.. code-block:: css

    html,body,div,span,applet,object,iframe,h1,h2,h3,h4,h5,h6,p,blockquote,pre,a,abbr,acronym,address,big,cite,code,del,dfn,em,img,ins,kbd,q,s,samp,small,strike,strong,sub,sup,tt,var,b,u,i,center,dl,dt,dd,ol,ul,li,fieldset,form,label,legend,table,caption,tbody,tfoot,thead,tr,th,td,article,aside,canvas,details,embed,figure,figcaption,footer,header,hgroup,menu,nav,output,ruby,section,summary,time,mark,audio,video{border:0;font-size:100%;font:inherit;vertical-align:baseline;margin:0;padding:0}article,aside,details,figcaption,figure,footer,header,hgroup,menu,nav,section{display:block}body{line-height:1}ol,ul{list-style:none}blockquote,q{quotes:none}blockquote:before,blockquote:after,q:before,q:after{content:none}table{border-collapse:collapse;border-spacing:0}

    body { line-height: 1;font-family: Arial, Helvetica, sans-serif;font-size: 12px; width: 100%; height: 100%; color: #000; font-size: 14px; }
    .clear { clear: both; }

    #wrapper { margin: 10px auto; width: 1000px; }
    #wrapper a { text-decoration: none; color: #F48A00; }
    #wrapper span.highlight { color: #F48A00; }

    #header { border-bottom: 1px solid #ccc; margin-bottom: 20px; }
    #header .top { border-bottom: 1px solid #ccc; margin-bottom: 10px; }
    #header ul.navigation { list-style: none; text-align: right; }
    #header .navigation li { display: inline }
    #header .navigation li a { display: inline-block; padding: 10px 15px; border-left: 1px solid #ccc; }
    #header h2 { font-family: 'Irish Grover', cursive; font-size: 92px; text-align: center; line-height: 110px; }
    #header h2 a { color: #000; }
    #header h3 { text-align: center; font-family: 'La Belle Aurore', cursive; font-size: 24px; margin-bottom: 20px; font-weight: normal; }

    .main-col { width: 700px; display: inline-block; float: left; border-right: 1px solid #ccc; padding: 20px; margin-bottom: 20px; }
    .sidebar { width: 239px; padding: 10px; display: inline-block; }

    .main-col a { color: #F48A00; }
    .main-col h1,
    .main-col h2
        { line-height: 1.2em; font-size: 32px; margin-bottom: 10px; font-weight: normal; color: #F48A00; }
    .main-col p { line-height: 1.5em; margin-bottom: 20px; }

    #footer { border-top: 1px solid #ccc; clear: both; text-align: center; padding: 10px; color: #aaa; }

Bundle Template - Level 2
.........................

We now move onto creating the layout for the Blog bundle. Create a file located at
``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` and add the
following content.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}

At a first glance this template may seem a little simple, but its simplicity is
the key. Firstly it extends the applications base template that we created earlier.
Secondly it overrides the parent sidebar block with some dummy content. As the
sidebar will be present on all pages of our blog it makes sense to perform the
customisation at this level. You may ask why don't we just put the customisation
in the application template as it will be present on all pages. This is simple,
the application knows nothing about the Bundle and shouldn't. The Bundle should
self contain all its functionality and rendering the sidebar is part of this
functionality. OK, so why don't we just place the sidebar in each of the page
templates? Again this is simple, we would have to duplicate the sidebar each
time we added a page. Further this level 2 template gives us the flexibility in
the future to add other customisations that all children templates will inherit.
For example, we may want to change the footer copy on all pages, this would be a
great place to do this.

Page Template - Level 3
.......................

Finally we are ready for the controller layout. These layouts will commonly be
related to a controller action, i.e., the blog show action will have a
blog show template.

Lets start by creating the controller for the homepage and its template. As this
is the first page we are creating we need to create the controller. Create the
controller at ``src/Blogger/BlogBundle/Controller/PageController.php`` and add
the following:

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class PageController extends Controller
    {
        public function indexAction()
        {
            return $this->render('BloggerBlogBundle:Page:index.html.twig');
        }
    }

Now create the template for this action. As you can see in the controller action
we are going to render the Page index template. Create the template at
``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig``

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        Blog homepage
    {% endblock %}

This introduces the final template format we can specify. In this example
the template ``BloggerBlogBundle::layout.html.twig`` is extended where
the ``Controller`` part of the template name is ommitted. By excluding the
``Controller`` part we are specifiying the use of the Bundle level template
created at ``src/Blogger/BlogBundle/Resources/views/layout.html.twig``.

Now lets add a route for our homepage. Update the Bundle routing config located
at ``src/Blogger/BlogBundle/Resources/config/routing.yml``.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /
        defaults: { _controller: BloggerBlogBundle:Page:index }
        requirements:
            _method:  GET

Lastly we need to remove the default route for the Symfony2 welcome screen.
Remove the ``_welcome`` route at the top of the ``dev`` routing file located at
``app/config/routing_dev.yml``.

We are now ready to view our blogger template. Point your browser to
``http://symblog.dev/app_dev.php/``.

.. image:: /_static/images/part_1/homepage.jpg
    :align: center
    :alt: symblog main template layout

You should see the basic layout of the blog, with
the main content and sidebar reflecting the blocks we have overridden in the relevant
templates.

The About Page
--------------

The final task in this part of the tutorial will be creating a static page for the
about page. This will demonstrate how to link pages together, and further enforce the
Three Level Inheritance approach we have adopted.

The Route
~~~~~~~~~

When creating a new page, one of the first tasks should be creating the route for it.
Open up the ``BloggerBlogBundle`` routing file located at
``src/Blogger/BlogBundle/Resources/config/routing.yml`` and append the following routing
rule.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_about:
        pattern:  /about
        defaults: { _controller: BloggerBlogBundle:Page:about }
        requirements:
            _method:  GET

The Controller
~~~~~~~~~~~~~~

Next open the ``Page`` controller located at
``src/Blogger/BlogBundle/Controller/PageController.php`` and add the action
to handle the about page.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        //  ..

        public function aboutAction()
        {
            return $this->render('BloggerBlogBundle:Page:about.html.twig');
        }
    }

The View
~~~~~~~~

For the view, create a new file located at
``src/Blogger/BlogBundle/Resources/views/Page/about.html.twig`` and copy in the
following content.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/about.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}About{% endblock%}

    {% block body %}
        <header>
            <h1>About symblog</h1>
        </header>
        <article>
            <p>Donec imperdiet ante sed diam consequat et dictum erat faucibus. Aliquam sit
            amet vehicula leo. Morbi urna dui, tempor ac posuere et, rutrum at dui.
            Curabitur neque quam, ultricies ut imperdiet id, ornare varius arcu. Ut congue
            urna sit amet tellus malesuada nec elementum risus molestie. Donec gravida
            tellus sed tortor adipiscing fringilla. Donec nulla mauris, mollis egestas
            condimentum laoreet, lacinia vel lorem. Morbi vitae justo sit amet felis
            vehicula commodo a placerat lacus. Mauris at est elit, nec vehicula urna. Duis a
            lacus nisl. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices
            posuere cubilia Curae.</p>
        </article>
    {% endblock %}

The about page is nothing spectacular. Its only action is to render a template file
with some dummy content. It does however bring us on to the next task.

Linking the pages
~~~~~~~~~~~~~~~~~

We now have the about page ready to go. Have a look at ``http://symblog.dev/app_dev.php/about``
to see this. As it stands there is no way for a user of your blog to view the about page,
short of typing in the full URL just like we did. As you'd expect Symfony2 provides both
sides to the routing equation. It can match routes as we have seen, and can also
generate URLs from these routes. You should always use the routing functions provided
by Symfony2. Never in your application should you be tempted to put the following.

.. code-block:: html+php

    <a href="/contact">Contact</a>

    <?php $this->redirect("/contact"); ?>

You may be wondering what's wrong with this approach, it may be the way you always
link your pages together. However, there are a number of problems with this approach.

1. It uses a hard link and ignores the Symfony2 routing system entirely. If you wanted to change
   the location of the contact page at any point you would have to find all references to the hard
   link and change them.
2. It will ignore your environment controllers. Environments is something we haven't really explained yet
   but you have been using them. The ``app_dev.php`` front controller provides us access to our application
   in the ``dev`` environment. If you were to replace the ``app_dev.php`` with ``app.php`` you will be
   running the application in the ``prod`` environment. The significance of these environments will
   be explained further in the tutorial but for now it's important to note that the hard link
   defined above does not maintain the current environment we are in as the front controller is
   not prepended to the URL.

The correct way to link pages together is with the ``path`` and ``url`` methods provided by Twig. They are
both very similar, except the ``url`` method will provide us with absolute URLs. Lets
update the main application template located at ``app/Resources/views/base.html.twig`` to link
to the about page and homepage together.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="#">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

Now refresh your browser to see the Home and About page links working as expected. If you view the source
for the pages you will notice the link has been prefixed with ``/app_dev.php/``. This
is the front controller I was explaining above, and as you can see the use of ``path`` has maintained
it.

Finally lets update the logo links to redirect you back to the homepage. Update the
template located at ``app/Resources/views/base.html.twig``.

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <hgroup>
        <h2>{% block blog_title %}<a href="{{ path('BloggerBlogBundle_homepage') }}">symblog</a>{% endblock %}</h2>
        <h3>{% block blog_tagline %}<a href="{{ path('BloggerBlogBundle_homepage') }}">creating a blog in Symfony2</a>{% endblock %}</h3>
    </hgroup>
    
Conclusion
----------

We have covered the basic areas with regards to a Symfony2 application including getting
the application configured and up and running. We have started to explore the fundamental concepts
behind a Symfony2 application, including Routing and the Twig templating engine.

Next we will look at creating the Contact page. This page is slightly more involved than the About page
as it allows users to interact with a web form to send us enquiries. The next chapter will introduce
concpets including Validators and Forms.
