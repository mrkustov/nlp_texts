## Классификация текстовых новостей. На основе текстового представления word2vec.

### Содержание:
1. Тетрадка с кодом **(nlp_text.ipynd)**
2. Русскоязычные обученные корпуса: 
    * news_upos_skipgram_300_5_2019; Русскоязычные новости; 2.6 миллиарда слов; *Universal Tags* **(184.zip)**
    * ruwikiruscorpora_upos_skipgram_300_2_2019; НКРЯ и Википедия за декабрь 2018; 788 миллионов слов; *Universal Tags* **(182.zip)**
3. Корпус для расстановки тегов (преобразование в « слово_его часть речи) **(udpipe_syntagrus.model)**
4. Текст рассказа *О. Генри "Русские соболя"* **(henry_soboly.txt)**
5. Таблица новостей с lenta.ru **(lenta-ru-news.csv)**
6. Таблица с предобработаными новостями **(good_text_teg.csv)**

Датасеты >2Гб и русскоязычные корпуса w2v >600Мб распологаются на [Гугл-диске](https://drive.google.com/drive/folders/1q1dbX2PJzbIbgO0eKxap9dNHtkMdEgKO?usp=sharing)


### Описание алгоритма классификации.
#### Первоначальная задача
Изначально стояла задача, кластеризации текстовых новостей. 
Был предложен алгоритм: 
1. Предпроцесс с текстовыми новостями
    * Лемматизация 
    * Приведение всех слов к нижнему регистру 
    * Удаление стоп-слов 
    * Присвоение меток (тегов) (преобразование в «слово_его часть речи»)
2. Преобразование слов в вектора. На основе текстового представления word2vec.
Для данной задачи, мы выбираем уже обученную модель (с сайта https://rusvectores.org/ru/models/) . Модель обучена на русскоязычных новостях
3. Преобразование новостей в вектора.

Каждая новость – вектор. Преобразование происходит усреднением векторов из которых состоит новость.
4. Кластеризация векторов.

Выделение кластеров при помощи метода k-means (с помощью библиотеки ntk, для кластеризации векторов). Поскольку сходство обычно измеряется с помощью косинусного расстояния, необходимо использовать косинусное расстояние, вместо Евклидового расстояния.

5. Выбор количества кластеров.

Выбор количества кластеров, путём максимизирования метрики качества кластеризации.

**Силуэт**

Данный коэффициент не предполагает знания истинных меток объектов и позволяет оценить качество кластеризации, используя только саму (неразмеченную) выборку и результат кластеризации. Сначала силуэт определяется отдельно для каждого объекта. Обозначим через – среднее расстояние от данного объекта до объектов из того же кластера, через – среднее расстояние от данного объекта до объектов из ближайшего кластера (отличного от того, в котором лежит сам объект).

Тогда силуэтом данного объекта называется величина:

Силуэтом выборки называется средняя величина силуэта объектов данной выборки. Таким образом, силуэт показывает, на сколько среднее расстояние до объектов своего кластера отличается от среднего расстояния до объектов других кластеров. Данная величина лежит в диапазоне . Значения, близкие к -1, соответствуют плохим (разрозненным) кластеризациям, значения, близкие к нулю, говорят о том, что кластеры пересекаются и накладываются друг на друга, значения, близкие к 1, соответствуют "плотным", четко выделенным кластерам. Таким образом, чем больше силуэт, тем более четко выделены кластеры, и они представляют собой компактные, плотно сгруппированные облака точек.

С помощью силуэта можно выбирать оптимальное число кластеров (если оно заранее не известно) – выбирается число кластеров, максимизирующее значение силуэта. В отличие от предыдущих метрик, силуэт зависит от формы кластеров и достигает больших значений на более выпуклых кластерах, получаемых с помощью алгоритмов, основанных на восстановлении плотности распределения.

7. Выбор названия кластеров.

Из новостей, которые попали в данный кластер будут отобраны 5 слов с наибольшей частотой встречаемости.

Название кластера – перечисление 5 наиболее встречаемых слов, через запятую.

8. Оценка методом человеческого восприятия.

Для начала посмотреть, схожесть слов в теме кластера. Все 5 слов должны быть связаны между собой. Если имеются не связанные слова, то необходимо посмотреть на новости, которые попали в данный кластер. Выяснить, почему это произошло? Путём настройки гиперпараметров добиться наилучшего результата.

#### Задача классификации новостей.
Далее, задача была переформулирована в задачу классификации текстовых новостей. Было предложено реализовать данный алгоритм на размеченных данных.
(P. S. предполагаю, что задачу кластеризации можно свести к задаче с частичным обучением с предварительным определением конечного числа кластеров(классов) и первоначальной разметной данных)

##### Алгоритм классификации
Алгоритм классификации до третьего пункта аналогичен. 

4 Разбиение первоначальных данных на "обучение" и "тест"

5 Классификация текстовых новостей методом «ближайшего соседа» (KNN) используя косинусного расстояние для выборки.

6 Построения сетки и подборки оптимальных значений параметров модели : «числа ближайших соседей».

7 Построение модели классификации на основе метода опорных векторов (SVM) для выборки.

8 Построения сетки и подборки оптимальных значений параметров модели: выбор ядра и подборка константы «С».

##### Примечание: В ходе выполнения работы использовались варианты многопоточного расчёта и обучения, а также бэггинга для повышения точности модели и снижения времени обучения.
