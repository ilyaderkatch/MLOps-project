# News summary

Описание проекта, цель которого создание модели, которая по новости создает краткое описание того, чему посвящена новостная статья.

## Формулировка задачи
Задача суммаризации новостей является частным случаем задачи суммаризации текста. Данная задача имеет несколько применений в реальной жизни. 

Во-первых, очень часто для понимания смысла статьи не хватает самого заголовка, однако сам текст статьи может быть весьма большим. С точки зрения читателя, подобные автоматически созданные краткие содержания позволят тратить меньше времени на чтение новостей. Кроме того, читатель, ищущий что-то конкретно в массиве статей, может ориентироваться на построенные автоматически краткие содержания для того, чтобы читать весь массив статей. Подобная практика распространена, например, в научных статьях: перед основным текстом идет abstract, ознакамливающий читателя с основными тезисами работы. С точки зрения создателя/агрегатора контента, подобная фича может сделать его более конкурентным по сравнению с остальным игроками на рынке. 

Во-вторых, модель, создающая краткое содержание статей, может использоваться для дальнейших исследований. Например, интересна задача кластеризации новостей по инфоповодам. Использование самих текстов статей является избыточным, так как классификатор может переобучаться под несущественные части статей. В то же время использование только заголовков является недостаточной стратегией. Кроме того, подобная кластеризация может использовать агрегаторами новостей для того, чтобы объединять различные источники, подающие одну новость, для читателя.

## Описание доступных датасетов

**XSum** --- набор данных, состоящий из 227 тысяч статей BBC
за период с 2010 по 2017 год, охватывающих широкий спектр тем, наряду
с профессионально написанными резюме из одного предложения. Минусы: суммаризация состоит только из одного предложения, что недостаточно для нашей задачи.

**CNN/DailyMail** --- набор данных, содержащий
93 тыс. статей из CNN и 220 тыс. статей
из газет Daily Mail. Оба издателя дополняют свои статьи краткими аннотациями. Плюсы: является классическим датасетом в задаче суммаризации. Минусы: только на английском языке. 

**Reddit TIFU** --- набор данных, содержащий 120 тыс. сообщений
с неофициальными историями с онлайн-дискуссионного форума Reddit,
более конкретно с субреддита TIFU за период с января 2013 по
март 2018. Публикации в субреддите строго следуют правилу
написания описательного резюме ”TL; DR” и имеют более высокое качество, чем (в котором использовалось больше субреддитов), основываясь на нашей ручной проверке. Плюсы: большое количество данных, созданных большим количеством людей, с различной тематикой. темы приближены к новостной повестке. 

**arXiv, PubMed** --- это два больших датасета для данных научных публикаций из arXiv.org (113k) и
PubMed (215 тыс.). Задача состоит в том, чтобы сгенерировать аннотацию из основного текста статьи. Минусы: суммаризация большого количества текста в каждой статье, что нетипично для новостных репортажей + специфическая тематика статей. 

## Подход к моделированию

Существует два подхода по построению выжимки текста: экстрактивный и абстрактный. Экстрактивная суммаризация подразумевает, что модель выделяет наиболее важные предложения текста, при этоим не изменяя их структуры. С одной стороны, это ограничивает качество модели, так как модель не может выделить только важные части предложения, а также ограничена тем, что не может объединять смысл разных предложений в одно. С другой стороны, данный подход позволяет создать более легковесные модели, часть из которых основана исключительно на статистических подходах, не применяя методы машинного обучения. В то же время абстрактная суммаризация гораздо ближе к привычному краткому содержанию текста, так как подобная суммаризация не ограничена выделением предложений из исходного текста. Цель подобного типа суммаризации максимально приблизиться к результату, который мог бы выполнить человек. В то же время, подобные модели гораздо сложнее для построения, а также для обучения. Однако с появлением архитектуры внимания и трансформеров, данный подход стал более доступный для разработки. В данный момент существует три лидирующие модели-суммаризаторы, которые основываются на данной архитектуре, ниже представлены их описания. Важно понимать, что все они обучены не на задаче суммаризации текста, а на задаче восстановления текста по его испорченному (с пропусками, перемешанными предложениями итд) аналогу. Данное обучение на более "абстрактных" задачах позволяет модели узнать устройство языка, что в дальнейшем позволяет применить подобные модели в задаче суммаризации.

**BART** ---  sequence-to-sequence Трансформер, который предобучается реконструкции испорченного зашумлённого текста. Разработка Facebook AI Research. На входе у модели каким-то образом испорченный текст, а на выходе ей надо сгенерировать его оригинальную версию. BART предобучался на том же корпусе, что и RoBERTa, то есть на составном корпусе из 4 частей: книг, новостей, документов по ссылкам из постов Reddit, кусочка Common Crawl.
Важно, что в отличие от BERT, эта модель сразу предобучается на генерации текста, и поэтому лучше подходит для автоматического реферирования. Наличие в задачах предобучения перемешиваний предложений и "вращений" документов также помогает при дальнейшем дообучении.

**T5** — ещё один sequence-to-sequence Трансформер, глобально аналогичный BART, разработка Google Research. T5 предобучается на задаче восстановления промежутков, то есть наборов подряд идущих токенов. При обучении промежутки исходного текста скрываются, и задача модели их сгенерировать. В отличие от BART, где текст генерируется целиком, T5 нужно сгенерировать только сами скрытые промежутки. Есть и некоторые небольшие архитектурные различия, а именно:

* LayerNorm перед сложением. В классическом Трансформере используется LayerNorm, который применяется после сложения с остаточными связями с предыдущего слоя. В T5 LayerNorm применяется до сложения.
* Упрощённый LayerNorm. В самом блоке LayerNorm используется масштабирование (scale), но не используется смещение (bias).
* Относительные позиционнные эмбеддинги. В классическом Трансформере позиционные эмбеддинги фиксированы и складываются со входами модели.

**PEGASUS** — модель со специально подобранной под автоматическое реферирование задачей предобучения. С точки зрения архитектуры это обычный sequence-to-sequence Трансформер, как и предыдущие две модели. Но вместо восстановления случайных кусков текста предлагается использовать задачу генерации пропущенных предложений. Она заключается в том, что мы выбираем самые важные предложения из документа, заменяем их на токен-маску, формируем из них квазиреферат, и пытаемся этот квазиреферат сгенерировать. Эту задачу можно решать одновременно в паре с маскированным языковым моделированием. Авторы предлагают 3 основных стратегии выбирать важные предложения: случайно; брать первые несколько предложений; брать несколько предложения по некой мере важности. В качестве меры важности можно брать похожесть предложений на оставшийся текст по ROUGE или даже жадно набирать квазиреферат аналогично методу "оракула" из предыдущей статьи.

Используя данные модели, предлагается дообучить их под конкретную задачу суммаризации новостей. Однако планируется дообучение не всех весов, так как это чрезвычайно трудозатратно, а дообучении дополнительной модели, чей ответ, сложенный с ответом основной модели, будет финальным ответом для задачи суммаризации. Данный подход называется LoRa, и я буду использовать его. 

## Способ предсказания

Если говорить про отдельную редакцию, то во внутреннем средстве для выкладывания материалов, будет возможность сгенерировать краткое содержание статьи, и, при желании, отредактировать предложенный текст непосредственно автором, который выкладывает новости. Если говорить про новостные агрегаторы, то данная модель может быть встроена в пайплайн после сбора новостей в Интернете, но до выкладывания их на ресурсе. Что-то больше сказать про это не могу, потому что не знаю в деталях, как раотают агрегаторы новостей, и как конкретно туда можно вставить использование модели. 


