# ГЛАВА 14. ПРЕДСТАВЛЕНИЕ СЛОВ (OMER LEVY)

## 14.1 Введение
Слова естественного языка можно рассматривать как дискретные символы, которые (за исключением некоторых морфологических признаков) [имеется в виду сходство однокоренных слов и т.п.] не имеют укорененной меры сходства в закрепленном за ними символьном представлении. Тем не менее, люди, как правило, способны классифицировать определенные слова как более похожие на другие. К примеру, *дельфин*, пожалуй, имеет больше сходства с *китом*, чем со *спагетти*, поскольку и дельфины и киты относятся к одному отряду китообразных. В то же время, [понятия] *дельфин* и *океан*, очевидно, теснее связаны друг с другом, чем со *спагетти*, поскольку, они оба являются концептами морской предметной области. С вычислительной точки зрения, нам бы хотелось иметь возможность измерять эти различные понятия подобия в надежде использовать их в системах обработки естественного языка [NLP].

Возможно,  наиболее признанным подходом для схватывания сходства слов является представление каждого слова из словаря в виде вектора в некотором непрерывном пространстве. Векторы имеют естественные операторы для выражения степени подобия, такие как **евклидово расстояние** [иначе: евклидова норма] и **косинусное сходство** [иначе: косинусное расстояние, косинусная мера – в дистрибутивной семантике, т.е. косинус угла между двумя векторами], которые обеспечивают возможность получить числовое значение [степени сходства] для любой заданной пары векторов. [Функция cos(α) выбрана для отражения степени подобия из того соображения, что, чем меньше угол между векторами, тем больше между ними сходства, т. к. cos(0) = 1]. Таким образом, данный подход направлен на назначение словам векторов таким образом, чтобы степень подобия двух слов можно было вычислить как функцию соответствующих им двух векторов. Эти векторы [в дистрибутивной семантике – контекстные векторы], обычно называют **эмбеддингами** [дословно: вставками, вложениями, имплантами слов; не нашел подходящего удачного перевода на русский], поскольку они создаются путем встраивания [в теоретико-множественном смысле – «отображения на»] выбранного словаря в [некоторое векторное пространство] Rⅾ (где d - размерность этого векторного пространства). В данной главе, для ясности, мы будем называть их ***векторами слов**.

Использование векторов слов дает нам три основных преимущества: эффективность [при манипуляциях с ними], а также возможности для обобщения и интеграции. Чтобы понять, почему вычисление степени сходства с помощью векторов эффективнее с точки зрения использования компьютерной памяти, нам следует рассмотреть альтернативу в виде явного хранения в памяти значений степеней подобия для каждой выбранной пары слов. Такой наивный подход потребует объема памяти пропорционального  квадрату размера словаря. С другой стороны, векторы слов могут иметь малую размерность (быть «разреженными»), требуя, в любом случае, лишь доли памяти [от требуемой при наивном подходе]. Использование векторов слов также накладывает ограничение транзитивности на отношения сходства, что упрощает процедуру обобщения. Интуитивно понятно, что если слово x похоже на слово y, а слово y похоже на слово z, то x и z не могут быть слишком непохожими.

Однако главное преимущество представления слов в виде векторов заключается не в непосредственном вычислении степеней их подобия. Векторы слов, как правило, используются в качестве входного слоя для глубоких нейронных сетей, где небольшое возмущение на уровне входных данных (например, замена слова его синонимом) не должно оказывать существенного влияния на выходные данные. Другими словами, сходные входные данные должны порождать сходные выходные данные. Возможность интегрировать векторы слов в сложную модель, используя при этом их неявное сходство, является, пожалуй, одним из основных факторов, позволяющих нейронным сетям обобщать лучше, чем традиционные статистические методы с богатым набором функций.

Основная парадигма получения векторов слов общего назначения основана на **Дистрибутивной гипотезе** (Harris 1954; Firth 1957), которая гласит, что слова, встречающиеся в схожих контекстах, как правило, имеют схожие значения. В результате этого лингвистического наблюдения было предложено множество вычислительных методов, тонкие (но важные) различия которых обсуждаются в разделе 14.2. Эти методы позволяют создавать «разреженные» многомерные векторы, которые затем преобразуются в плотные, маломерные представления с помощью методов снижения размерности (раздел 14.3). Далее мы обсудим другие методы получения векторов слов, которые выходят за рамки вычислительной парадигмы **Дистрибутивной гипотезы** (раздел 14.4). Наконец, мы рассмотрим контрольные показатели для оценки качества векторов и производимых на их основе функций для вычисления степени подобия (раздел 14.5). 

## 14.2 Дистрибутивная гипотеза
Харрис (1954) заметил, что разница в значении двух слов тесно связана с различием в распределении их контекстов, т.е. тех словесных окружений, в которых каждое слово имеет тенденцию появляться. И наоборот, слова, которые встречаются в похожих контекстах, как правило, имеют схожие значения. Ферт (1957) афористично сформулировал это правило, сказав, что «вы узнаете слово по его компании» [не исключаю аллюзию на крылатое латинское выражение: «Ex ungue leonem», но с противоположным смыслом: не по части узнаем целое, а по целому узнаем часть].

В качестве примера того, как много информации может предоставить даже небольшой контекст, рассмотрим следующее предложение:

    «Вам следует полить немного *амбы* на свой фалафель, это вкусно!»

Хотя большинство читателей не знакомы с термином *амба*, они, вероятно, сделают вывод о том,  что это ближневосточная приправа, хотя она никогда не была определена явно. Этот пример демонстрирует интерпретацию Дистрибутивной гипотезы Фертом.

Однако, Ферт в своей интерпретации  делает одно очень сильное предположение: читателю уже должно быть известно, что означают другие слова, образующие контекст. [Чтобы угадать значение слова *амба*, нужно, по крайней мере, знать, что фал`афель – это ближневосточная закуска из бобовых].  И, поскольку, это предположение слишком многого требует с вычислительной точки зрения, мы обратимся к более деликатной формулировке Харриса. Вместо того чтобы пытаться определить значение одного отдельно взятого слова, Дистрибутивная гипотеза предоставляет инструментарий для измерения степени сходства между парами слов. Таким образом, что векторы [слов], полученные в результате применения данного инструментария должны всегда интерпретироваться не изолированно, а в сравнении с другими векторами заданного [векторного] пространства. Такую процедуру можно производить явно, вычисляя степень сходства между двумя векторами или неявно, используя векторы из данного векторного пространства для кодирования входных данных некоторой нейронной сети.

Чтобы продемонстрировать, как работает описанная идея,  мы используем наиболее простое определение контекста. Будем считать, что контекст задают соседние слова, которые появляются в пределах нескольких лексем [или токенов] от целевого слова множество раз в некотором текстовом корпусе. Тогда мы  сможем представить слова «дельфин» и «кит» в виде следующих наборов (двоичных векторов [на вычислительном уровне абстракции]) контекстообразующих слов:

    дельфин = {млекопитающее, море, океан, дыхало, афалина, умный, ...}
    кит = {млекопитающее, море, океан, дыхало, горбатый, огромный, ...}

Легко заметить, что эти два набора в значительной степени перекрываются, тогда как аналогичное представление слова «спагетти», вероятно, не будет иметь столько же общих контекстообразующих слов.

Вычислительная парадигма, основанная на Дистрибутивной гипотезе, часто **называемая дистрибутивной семантикой** или **дистрибутивным подобием**, допускает и гораздо более сложные представления слов, чем те, что приведены выше. В рамках этой парадигмы мы сначала определяем набор данных [датасет] D, как множество пар слов (w,c) из некоторого контекста. Этот набор данных порождает как словарь **целевых слов** (Vw), т.е. таких слов, для которых мы хотим задать их векторные представления, так и словарь **контекстов** (Vс), который, по сути, снабжает нас характеристиками, которые мы используем для описания целевых слов. Мы можем определять контексты различными способами: задавать отдельными словами или предложениями, документами или даже использовать неязыковые элементы, такие как изображения (см. раздел 14.2.1).

Каждая пара (w,c) из [набора данных] D отражает частоту совместного появления экземпляра слова w и экземпляра контекста c. Эти частоты совместных появлений обычно представляют в виде разреженной матрицы [дословный перевод, возможно, есть специальная терминология] Mco ∈ ℝ|Vw|x|Vc|, размерность строк которой соответствует количеству целевых слов, а размерность столбцов - количеству контекстов. [Другими словами, каждая строка является вектором лингвистической единицы (слова, словосочетания), координаты которого представляют собой числа, показывающие сколько раз данная лингвистическая единица встретилась в данном контексте. Каждый столбец соответствует координатному измерению вектора и интерпретируется как контекст]. Эта матрица, известная как **матрица совместного появления контекста и слова** [дословный перевод] затем, как правило, дополнительно обрабатывается для получения [более компактных] векторов слов (раздел 14.2.2). Далее обработанные строки матрицы [с переоцененными частотами] используются как вектора слов. Наконец, один из различных операторов над векторным пространством может быть использован для вычисления сходства между двумя векторами слов (раздел 14.2.3). [Как правило, в качестве такого оператора выбирается оператор расчета косинусной меры].