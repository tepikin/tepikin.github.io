---
layout: post
title:  "Android. Защита приложений от взлома"
tagline:  "Защита приложений от взлома."
date:   2016-12-08 00:00:00
vkcomments: true
share_buttons: true
categories: main
---

<!--# Android. Защита приложений от взлома-->
<!--![](http://tepikin.github.io/images/android_pirat.png)-->

Всем известно, что полностью защититься от взлома на Android невозможно. Однако с целью защиты приложений можно существенно усложнить данный процесс. Для этого сперва нужно разобраться с тем, какие типы взлома бывают, чем они опасны, и что с
ними делать. Предлагаю сначала разобраться с тем, зачем вообще нужно защищать приложения от взлома, и далее плавно перейти непосредственно к реализации различных типов защит.
<cut />
<!--more-->

## Нужно ли защищать приложения от взлома?
Вопрос на самом деле не такой простой, как это может показаться на первый взгляд. В качестве аргумента попробую привести пример из личного опыта. 

Когда одно из наших приложений перешло порог 1 млн скачек, количество взломанных версий увеличилось в разы. Как мы об этом узнали? Достаточно было просто вбить в поисковике название нашего приложения, и появлялась куча сайтов с предложением бесплатно скачать Pro версию. Растащить Pro версию по сайтам это еще не взлом, скажете вы,- но у нас была встроена проверка LVL, которая должна защищать от подобных действий. Также находились статьи о том, как взломать наше приложение с помощью LuckyPatcher. Причем взламывали не только внутренние покупки и платную версию, были попытки взломать даже наше API: приходили параметры с неверной подписью, запросы на несуществующие методы, имелись попытки подобрать root пароль к серверу - что совсем уж смешно. Больше всего удивил 4pda - там есть топик, в котором приложение "потрошат" от и до, это не считая полностью разлоченной Pro версии.

Как бы это не показалось странно, но мы не предпринимали никаких действий, чтобы это прекратить. Хотя можно было связаться с соответствующими сайтами и попросить их удалить взломанные версии (большинство сайтов легко выполняют это требование). Причина этого крайне проста - подобное "теневое" распространение увеличивает популярность приложения и показы рекламы. Если пользователь нашел инструкцию по взлому или старательно искал уже взломанную версию, значит он не готов за неё заплатить. Такой пользователь может пользоваться взломанной версией, рекомендовать её друзьям и оставлять о ней хорошие отзывы (не на Google Play). Поскольку пользователь приложил некоторые усилия, чтобы получить взломанную версию, значит, такой пользователь более лоялен к приложению. Хотя тут кроются и минусы. Минус заключается в том что многие пользователи пользуются очень старыми версиями приложения, и не обновляются, боясь потерять "бесплатность". Они терпят постоянные креши, используют крайне урезанный функционал, и устаревший интерфейс. Тем самым пользователи снижают популярность, рассказывая какое приложение убогое и устаревшее.

Когда приложение приблизилось к порогу 5млн скачек, мы ожидали повторной волна взломов. Приложение уже было достаточно популярное, поэтому дополнительная реклама от "теневого" распространения не нужна, анти-реклама тем более. Мы начали готовит очередное крупное обновление со встроенной защитой. Как вы уже поняли, мой ответ на вопрос "Нужно ли защищаться от взлома приложений?" такой:

>
> Защита не нужна, пока это не мешает развитию проекта.
>

Замечу, что знать о проценте взломанных версий приложения всегда полезно, независимо от стадии развития проекта. Поэтому, советую всегда встраивать защиту, хотя бы в пассивном режиме. Чтобы просто отсылалась информация на сервер, без применения каких либо противодействий. 

## Какие типы взломов бывают?

В данном разделе постараюсь перечислить и очень кратко описать типы взлома, иначе этот раздел разрастется до размеров отдельной статьи. Еще раз уточню, что это именно методы взлома приложения, а не использование его уязвимостей (таких как публичные провайдеры, глобальные ресиверы и т.д.). Под взломом будем понимать как подделку (tampering) приложений, так и результаты декомпиляции. Итого, если делить на очень крупные группы, то получим следующее:

* _Выставление неизменного приложения в другом маркете_ - Приложение копируется из Google Play и выставляется в другом маркете как платное (продавать чужое добро всегда выгодно). В общем смысле это не взлом, однако, это деструктивные действия направленные против разработчика. С этими действиями можно и нужно бороться, поэтому данный вопрос также будет рассматриваться в этой статье.

* _Замена `packageName`_ - Используется для того чтобы приложение можно было выставить в том-же маректе, из которого его скачали,- цель перепродажа.

* _Замена `apiKey`_ рекламных площадок - Выделил в отдельный пункт, потому что применяется очень часто. Цель понятна, получать доход с рекламы в чужих приложениях. Кстати говоря, Admob пытается бороться с этим самостоятельно, но об этом позже.

* _Разлочивание покупок_ - Бесплатный доступ к платному контенту. В основном это делают сами пользователи, например, приложением Freedom. Однако часто встречаются ситуации, когда пропатченное (разлоченное) бесплатное приложение продают как Pro версию.

* _Получение доступа к скрытому Api_ - Цель очевидна (например для того, чтобы постить фотки в инстаграм). Также используется для фальсификации отправляемых на сервер данных, например, накрутка кликов в Admob. Рассматриваем именно тот вариант когда Api защищено, например, подписывается клиентом, и просто посмотреть запросы через прокси недостаточно.

* _Встраивание вредоносного кода_ - добавление к текущему коду дополнительного функционала. Функционал может быть самый различный от отправки платных SMS и поиска номеров кредиток, до встраивания дополнительной рекламы и создания всяческих ботнетов.

Есть еще множество разнообразных угроз, например, фишинговые приложения, но мы их рассматривать не будем. Рассмотрим только те угрозы, от которых приложение должно обороняться "само" в автоматическом режиме. Как мы видим, основных направлений не так уж много, поэтому можно будет поговорить подробнее о каждом в отдельности (хотя все они во многом пересекаются).

## Насколько легко взломать приложение?

Всем известны такие приложения как "Freedom" и "Lucky Patcher" - это Android приложения, которые делают доступными все покупки и снимают защиту LVL. Мне кажется, что это уже ответ на поставленный вопрос.

![Lucky Patcher](http://tepikin.github.io/images/android_pirat_lucky_patcher.png)

Хочу поделиться историей. Один мой знакомый, никак не связанный с IT, спросил меня: "Как выводить деньги с Amazon App Store". Я крайне удивился такому вопросу, и после уточнения подробностей, выяснилось следующее. Он скачивает приложения с Google Play, ломает их LuckyPatcher'ом (снимает LVL защиту, если она есть) и выставляет на Amazon. Недавно он нашел приложения для смены `packageName` и рекламных ключей для площадок Facebook и Admob. Таким образом, человек который вообще не умеет программировать, имеет массу приложений в маркетах и стабильный (хоть и нелегальный) доход со сторов и рекламы. Еще раз уточню, сам он не дкеомпилирует приложения, он использует программы которые осуществляют взлом за него в автоматическом режиме.

>
> Не защищенное приложение может взломать любой, кто захочет.
>

Все работает так легко, потому что защита приложения (напримел LVL) это вызов однотипных функций и для того, чтобы эту защиту убрать, достаточно выполнить замену по шаблону в `apk` файле (это грубое описание). Поэтому использование шаблонных компонентов кода это крайне плохо. Такие компоненты легко найти, лего получить ключи, которые всегда лежат в одинаковых местах и т.д. Поэтому ценность любой библиотеки для защиты apk со временем падает. 

Еще одна причина легкого взлома это сама Java (вернее байт код). Apk декомпилируется в Smali, который очень удобен для восприятия. Многие сравнивают Smali с Assembler и забывают добавить что это "Объектно-ориентированный Assembler". Лично я считаю, что это вполне дружелюбный (user friendly) язык программирования. Разобраться в нем ничуть не сложнее чем в любом другом. И соответственно изменить функционал с его помощью тоже не сложно.

Многие считают, что если вынести "ответственный" код в NDK, то его будет сложнее взломать. В некоторых случаях это действительно так. Но я считаю, что это наоборот облегчает злоумышленнику процесс взлома. Поясню на примере Instagram:

Для подписания каждого запроса используется секретный ключ. Раньше он был просто зашит в Java код и получался крайне просто. Затем его вынесли в strings.so и его можно было залогировать при обращении к данной библиотеке. А сейчас весь процесс подписания вынесен в .so. Казалось бы, способ подписи никак не узнать. Но смысл как раз в том, что нам и не нужно знать алгоритм. Теперь можно просто скопировать .so библиотеку в свой проект (или на сервер arm) и вызывать `StringBridge.getSignatureString(params)`. Стало еще удобнее. Вышла новая версия - скопировал библиотеки и всё готово. При таком подходе нужно весь процесс отправки запроса выносить в Ndk, а это уже слишком трудозатратно.

1 Сентября 2010 Свет увидел [статью](http://android-developers.blogspot.ru/2010/09/securing-android-lvl-applications.html) от Google про License Verification Library. В статье, конечно, сказано что нужно постоянно модифицировать код, относящийся к проверке лицензии, не выделять отдельные методы, а использовать inline code. Но проблема в самой библиотеке. Вернее в том, что она является библиотекой и, следовательно, шаблонным кодом. Её легко найти по `packageName  com.google.android.vending.licensing` и легко обойти. На эту тему и так много статей на хабре, поэтому не буду останавливаться на этом моменте. 

Про обфускацию. Конечно, она затрудняет чтение, но лишь в том случае если код читает человек. Если же происходит замена по шаблону (как делает Lucky Patcher), то неважно как именно называется класс "MySuperClass" или "AAB". Также всегда есть части кода, которые обфусцировать нельзя. Это такие методы как `onCreate()`, `OnClick()` или названия пакетов такие как `com/android/vending/licensing/` и т.д.. Еще не все умеют (или не хотят) обфусцировать код до конца. Например, facebook обфусцирует свой код весьма странно - открываем класс `com/facebook/common/a/b.smali` и видим его настоящее название:

``` smali
.class public interface abstract Lcom/facebook/common/a/b;
.super Ljava/lang/Object;
.source "LoggingDelegate.java"       //  <<-- real name

virtual methods
.method public abstract a()I
.end method

.method public abstract a(I)V
.end method
```

Несмотря на все эти недостатки, в целом, обфускация повышает сложность взлома. Как и каждый описанный выше метод, она добавляет "камешек" на пути к взлому. И чем больше таких камешков, тем больше вероятность, что злоумышленнику надоест спотыкаться, и он свернет с пути злодея (по крайней мере, мне хочется в это верить).

О том, как обстоят дела, мы немного узнали и теперь встает вопрос - что с этим делать?

## Как защититься?

С незащищенными приложениями мы разобрались. Обратим внимание на приложения, о защите которых подумали заранее. Как уже говорилось, защититься полностью невозможно и злодей всегда сможет делать всё то же самое, что может делать приложение. Однако усложнить процесс взлома можно. Это я понял на собственном опыте.

Однажды мне понадобилось залогировать некоторые данные из чужого приложения. Сразу оговорюсь, что это был не злой умысел. Просто приложение поддерживало шаринг, это я знал наверняка, но стандартный `Intent.ACTION_SEND` не поддерживало (но речь собственно не об этом). После декомпиляции приложения, я добавил брекпоинты во все интересующие меня методы, и запустил приложение. Приложение запустилось, но падало в совершенно случайных местах. Естественно, я подумал, что это я сделал что-то не так, и стал искать проблему. Сложность была в том, что для поиска не было никакой привязки. Падение не было связано ни с конкретным экраном, ни с действием пользователя, и даже стека ошибок в `logcat` или тоста (`Toast`) о том что что-то не так - не было. Спустя 4 часа я нашел следующее (не буду приводить smali они слишком громоздкие):

``` java
if (BuildConfig.DEBUG) postDelayed(() -> System.exit(0),10000l);
```

Всего одна строчка кода отняла у меня 4 часа. Подобная проверка в коде встречалась в трех местах. Естественно я просто удалил эти проверки и... ничего не изменилось. Спустя еще пару часов обнаружилось:

``` java
if (signInvalid) postDelayed(() -> sendBroadcast(new Intent(ACTION_CLOSE_ALL_ACTIVITIES)),10000l); 
```

Естественно это моя интерпретация кода, на самом деле action назывался `"655aefa70ea7e3338207978db6f7ebdc"`, и все `Activity`, которые были на него подписаны, просто закрывались по `finish()` (собственно это и помогло мне его найти).

Эта история несет единственную цель - показать что:

>
> Незначительные усилия со стороны обороняющихся, несут большие проблемы нападающим.
>

Я решил написать небольшую библиотеку, в которой будут собраны самые простые и эффективные методы защиты. Понимаю, что данная библиотека ценна, пока о ней никто не знает. Но рассматривайте её как "описание приемов". Всё сделано максимально просто и понятно, чтобы любой мог взять оттуда кусочек кода для себя. 

## Где же код ?

Приступим. Библиотека [AndroidTamperingProtection](https://github.com/tepikin/AndroidTamperingProtection). 

Для совсем не терпеливых, опишу простейший вариант использования. Копируем класс [TamperingProtection.java](https://github.com/tepikin/AndroidTamperingProtection/blob/master/tamperingprotection/src/main/java/ru/lazard/tamperingprotection/TamperingProtection.java) в свое приложение и пишем следующее:

``` java
TamperingProtection protection = new TamperingProtection(context);
protection.setAcceptedPackageNames("packageName вашего приложения"); 
protection.setAcceptedSignatures("Md5 подпись - fingerprint");
protection.validateAll(); // <- возвращает false если взломано, и true если все хорошо
``` 

А теперь давайте более подробно обо всех функциях библиотеки.

Сразу оговорюсь, что библиотеку писал специально для этой статьи и очень торопился, поэтому могут быть ошибки (не ругайте строго).

Чем же меня не устроили существующие библиотеки:

* Таких библиотек мало. Чем больше (пусть и однотипных) библиотек по защите, тем сложнее взломщикам. Им сложнее подобрать шаблон, о чем было сказано ранее.

* Многие из существующих библиотек сложны. Их нельзя прочесть полностью за пару минут и на их основе создать что-то новое. Данный код лучше модифицировать, а не копировать из проекта в проект.

* Библиотеки основанные на NDK. Брать готовую скомпилированную .so библиотеку не имеет никакого смысла, в плане защиты. А компилировать чужие С++ коды, да еще и модифицировать их от проекта к проекту отнимает некоторое время. А хорошая защита не должна отягощать разработчика, ведь взломать смогут в любом случае.

* В основном библиотеки содержат один или два метода проверки. Хочется собрать все варианты вместе, чтобы был выбор, даже если использоваться из них будет всего один. 

Библиотека предназначена для проверки на взломанность. Никаких действий, например аварийного закрытия приложения, она не совершает. Единственное её назначение это получение информации о том взломано приложение или нет, а как реагировать на полученные данные решать уже вам.

Встроить в свое приложение можно 3 способами. 

* __Самый правильный вариант__: посмотреть как устроены методы проверки apk, обдумать, и написать собственную реализацию.

* Просто скопировать класс [TamperingProtection.java](https://github.com/tepikin/AndroidTamperingProtection/blob/master/tamperingprotection/src/main/java/ru/lazard/tamperingprotection/TamperingProtection.java) в свое приложение, как уже было написано ранее. 

* Добавить в Gradle. Для этого в `build.gradle` корневой директории добавить 

``` gradle
allprojects {
   repositories {
       ...
       maven { url "https://jitpack.io" }
   }
}
//  в build.gradle файл проекта. Добавить зависимость
dependencies {
   compile 'com.github.tepikin:AndroidTamperingProtection:0.11'
}
```

Что же может проверять данная библиотека? Если посмотреть полный список проверок, то он выглядит так:

``` java 
// Keep dexCrc in resources (strings.xml) or in JNI code. Don't hardcode it in java classes, because it's changes checksum.
long dexCrc = Long.parseLong(this.getResources().getString(R.string.dexCrc)); 

TamperingProtection protection = new TamperingProtection(context);
protection.setAcceptedDexCrcs(dexCrc);
protection.setAcceptedStores(TamperingProtection.GOOGLE_PLAY_STORE_PACKAGE); // apps installed only from google play
protection.setAcceptedPackageNames("ru.lazard.sample.Lite_Version","ru.lazard.sample.Pro_Version"); // lite and pro package names
protection.setAcceptedSignatures("CC:0C:FB:83:8C:88:A9:66:BB:0D:C9:C8:EB:A6:4F:32"); // only release md5 fingerprint
protection.setAcceptStartOnEmulator(false); // not allowed for emulators
protection.setAcceptStartInDebugMode(false); // not allowed run in debug mode

protection.validateAllOrThrowException(); // detailed fail information in Exception.
```

`setAcceptedDexCrcs` - Проверка [CRC](https://ru.wikipedia.org/wiki/%D0%A6%D0%B8%D0%BA%D0%BB%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B8%D0%B9_%D0%B8%D0%B7%D0%B1%D1%8B%D1%82%D0%BE%D1%87%D0%BD%D1%8B%D0%B9_%D0%BA%D0%BE%D0%B4) для файла classes.dex. Предназначен он для того чтобы проверить был ли изменен исходный код приложения (Java классы). Метод хорош тем, что он надежен. Если CRC не совпал, то приложение точно менялось. Но у него есть один минус - его крайне неудобно использовать. Как вы уже поняли CRC код нельзя хранить в Java классах. Поскольку для получения этого кода нужно сначала скомпилировать приложение, а потом когда вы получите код и захотите зашить его в java класс, то вам придется опять перекомпилировать приложение и данный код изменится (dead lock). Поэтому данный код нужно хранить либо в ресурсах приложения, либо Ndk, либо на веб-сервере ( и это резко снижает эффективность защиты ). Чтобы узнать текущий CRC код можно воспользоваться методом `getDexCRC(context)` (не забудьте что он меняется при каждом изменении кода) или просто выполнить проверку с неверным кодом и в тексте ошибки будет написан текущий CRC код. Метод `getDexCRC(context)` устроен очень просто:

``` java
@NonNull
public static long getDexCRC(@NonNull Context context) throws IOException {
   ZipFile zf = new ZipFile(context.getPackageCodePath());
   ZipEntry ze = zf.getEntry("classes.dex");
   return ze.getCrc();
}
```

Как видите встроить подобную проверку - дело трех строк, а вероятность взлома понижается в разы.

`setAcceptedStores` - Проверка из какого маркета было установлено приложение. Поможет защититься, если злоумышленник опубликовал ваше приложение на другой площадке, или пользователь установил приложение сам не через маркет. Строго говоря, Google [не рекомендуте](http://android-developers.blogspot.ru/2010/09/securing-android-lvl-applications.html) использовать подобную проверку, объясняя это тем, что данная функция не документирована и т.д.. Но можно смело утверждать, что на текущий момент для Google Play данная проверка работает. Как же реализован данный метод:

``` java
public static String getCurrentStore(Context context) {
   return context.getPackageManager().getInstallerPackageName(context.getPackageName());
}
```

Как видите проверка тоже крайне лаконична. Сразу приведу `packageName` для наиболее известный маркетов : 

   * Google Play - `com.android.vending`
   * Amazon app - `com.amazon.venezia`
   * Samsung app - `com.sec.android.app.samsungapps`
   * Если пользователь установил сам - то вернется null

`setAcceptedPackageNames` - Проверка названия пакета вашего приложения. Защищает от повторного выкладывания вашего приложения в маркет. Данный метод рекомендую, потому что он прост и надежен. Проверить и сравнить с правильной версией можно так:

``` java
public static String getCurrentStore(Context context) {
   return context.getPackageManager().getInstallerPackageName(context.getPackageName());
}
```

`setAcceptedSignatures` - Проверяет подпись вашего приложения. Данный метод тоже рекомендую использовать. Единственное что меня смущает, это то что приходится где-то хранить отпечаток подписи ключа (хотя это уже паранойя).

Данная подпись зависит не от apk, а от ключа для подписи приложение. Отпечаток ключа можно получить, используя командную строку:

``` sh
keytool -list -v -keystore <YOU_PATH_TO_KEYSTORE> -alias <YOU_ALIAS> -storepass <YOU_STOREPASS> -keypass <YOU_KEYPASS> 
```

для debug версии соответственно:

``` sh
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android 
```

Или можно получить подпись текущего приложения, просто вызвав метод `TamperingProtection.getSignatures(context)`. Если будите реализовывать собственный алгоритм проверки подписи, то сообщу, что получить все подписи приложения можно так:

``` java
context.getPackageManager().getPackageInfo(context.getPackageName(), PackageManager.GET_SIGNATURES).signatures;
```

`setAcceptStartInDebugMode` - Из названия понятно, что это проверка на дебаг. Очевидно, что конечный пользователь не должен запускать приложение в режиме дебага, но мне этот метод не очень нравится, потому что его тяжело встроить. Ведь вы как разработчик будите запускать приложение в debug режиме. А делать проверку `if (isDebug) allowDebug();` это абсолютно бессмысленно. Поэтому код debug и release версий придется делать разным, а это очень плохо в плане тестирования.

`setAcceptStartOnEmulator` - Достаточно спорный метод. С одной стороны конечный пользователь не запускает приложение на эмуляторе. К тому же на эмуляторе нет маркета, и приложение туда вообще не должно попадать. Но с другой стороны автоматическое тестирование приходит на эмуляторах, есть возможность запускать приложение на десктопе (через расширения Chrome), да и злоумышленник может взламывать приложение на телефоне, а не эмуляторе. В общем можно сказать, что метод действенный, но неудобный. Проверка на то является ли устройство эмулятором взято из [этой](https://github.com/gingo/android-emulator-detector) библиотеки. Оно очень громоздкое, поэтому приводить я его не буду.

Из всех вышеперечисленных методов рекомендую использовать все для логирования и аналитики, просто чтобы у вас эта информация была. А для решительных действий, таких как принудительное завершение приложения, только `setAcceptedPackageNames` и `setAcceptedSignatures`. Поскольку они наиболее очевидны и легко реализуемы.

Еще хотел рассказать про метод `validateAllOrThrowException()`. Данный метод реализован на `handled exceptions` и в случае если валидация не прошла, то произойдет ошибка `ValidationException`, в которой будет детальное описание причины, по которой валидация не прошла. Это очень удобно, когда метод возвращает не просто `true` или `false`, а именно причину, например:

```
Not valid signature: CurrentSignature="CC:0C:FB:83:8C:88:A9:66:BB:0D:C9:C8:EB:A6:4F:33";  validSignatures="[CC:0C:FB:83:8C:88:A9:66:BB:0D:C9:C8:EB:A6:4F:32]";
```

Хорошо это или плохо можно спорить до бесконечности, поэтому для тех, кому этот подход не нравится, есть метод `validateAll()` который возвращает `true`/`false`.

## Заключение

Зачем же была написана эта статья? Конечно не только для того чтобы помочь тем кто столкнулся с подобной проблемой, и не для того чтобы привлечь внимание к данной проблеме в целом. А из самых корыстных целей. Хочется узнать, кто как защищает свои приложения, в общих словах, конечно же. 

* Как кто относится к защитам типа "встроенным мини антивирус" как это сделано у Сбербанка.
* Как защищают Api (не на уровне протокола, а именно защита самих алгоритмов шифрования).
* Встраивает ли кто-то защиту на основе DexOpt (подгружаемых библиотек). И если да то не противоречит ли это политике Google.

В общем, буду рад пополнить свой багаж знаний из комментариев к статье.

__Ps__ Уважаемые представители "Властных струкур". Никогда ничьи программы не взламывал и даже не декомпилировал, все персонажи и события вымышленные, исходники Smali сгенерированы путем случайного нажатия клавиш. Любые совпадения случайны. Вся информация исключительно в ознакомительных целях.
