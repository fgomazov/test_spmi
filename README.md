1. (test1.csv) Вам необходимо выполнить оценку публикационной деятельности сотрудников университета: 1.	Составить сводную таблицу о публикационной активности сотрудников университета, содержащую следующие столбцы:
    1.	Автор (scopus.csv)
    2.	ID (scopus.csv)
    3.	Количество публикаций в 2022 г. (scopus.csv)
    4.	Количество публикаций за 2020-2022 гг. (scopus.csv)
    5.	Количество публикаций Q1 и Q2 за 2020-2022 гг. (scopus.csv)
        -	Квартиль журнала (Q) можно найти на общедоступном портале scimago.com
    6.	Количество цитирований, полученных за 2021-2022 гг. (Scopus_Citation_Tracker.csv)
    7.	Количество упоминаний авторов Babyr и Tcvetkov в каждой статье из столбца пристатейных ссылок (scopus.csv).
2. (test2.xlsx) Проанализировать и охарактеризовать изменения в публикационной активности сотрудников за 2020-2022 г.

## 1. Подготовка данных через Excel
### Подготовка данных описана в файле 1_Prepare_Excel.md
#### Краткий алгоритм действий
##### 1. **Скачивание файлов**  scopus.csv и Scopus_Citation_Tracker.csv с репозитория текстового задания
##### 2. **Оценка и очистка файлов**
- Очистка файлов от пустых строк, шапки
- Оценка данных в файле и удаление лишних столбцов
##### 3. **Переименование столбцов** в файлах в единый формат, для удобства использования в Python через Jupiter Notebook
##### 4. **Обнаружено одинаковое количество строк** в указанных файлах, при помощи функции ЕСЛИ
##### 5. **Проведено сравнение по трем параметрам**. Данные совпали
- Год
- Название статьи
- Авторы
##### 6. **Объединенены файлы** при помощи копирования столбцов, с внесением изменений в случае различий в формате написания
##### 7. **Скачаны базы данных источников** за 2020 - 2022 года с сайта https://www.scimagojr.com/journalrank.php были 
##### 8. **Очищены от лишних столбцов** три базы scimaggojr_2020.csv, scimaggojr_2021.csv и scimaggojr_2022.csv, остались только
- Title : переименован в sourcee
- Type 
- SJR Best Quartile : переименован в quartile_2020 (2021, 2022)
### После данной подготовки получились следующие файлы, они приведены в репозитории
- spmi_scopus.csv
- scimaggojr_2020.csv
- scimaggojr_2021.csv
- cimaggojr_2022.csv

## 2. Обработка данных в Jupiter Notebook с использованием языка Python и библиотеки pandas
### Подробно работа приведена в файле 2_Python.md
### Jupiter Notebook с работой приложен в файле spmi_scopus.ipynb
#### Краткий алгоритм действий

##### 1. **Импорт библиотеки pandas**

##### 2. **Загрузка и предобработка датафрейма spmi из таблицы scopus.csv**
1. **Обработка данных** в ячейках. Использовались методы: str.rstrip(';'),  .duplicated().sum() .isnull().sum() .fillna(0)
2. **Разделение данных в столбцах** authors, IDs и affiliation по разделителям, удаление пробелов в начале ячейки и записывание итога в датафреймы. . Использовались методы .assign .str.split(',') .str.strip() .explode('column'), при проверке путем print(df.shape) было определено что общее количество строк в трех датафреймах не одинаково
3. **Выяснение причин несовпадения** - были скачаны три таблицы из трех датафреймов и проанализированы в Excel через ЕСЛИ, в аффилиации к статье  - Problems of Applying Pb-Free Technology in Soldering Electronic Components был обнаружен знак ";" использованный ранее для разделения на строки, в связи с чем, появились три дубля со следующей аффилиацией без указания автора:  'Department of Electronic Systems, Saint Petersburg, Russian Federation'. Изменения были внесены в исходный файл spmi_scopus.csv, поскольку ошибка только для одной статьи и важно сохранить аффилиацию. При повторной проверке количество строк в трех датафреймах совпало и составило 6575 при 11 столбцах
4. **Создание результирующего датафрейма** путем превращения столбцов ID и affiliation в список идентификаторов. Использовалась лямбда функция для создания списка и формирования новых колонок. В результате появился датафрейм состоящий из последовательных строк разъединенных элементов в столбцах authors, IDs и affiliation.
5. **Определение авторов с аффилиацией в СПГУ** При помощи двойного последовательного применения метода .contains с аргументами ('Mining) и ('Petersburg')
6. **Сформирован датафрейм** scopus с данными сотрудников СПГУ - удалены дубликаты .drop_duplicates(), удаление лишних столбцов
7. **Определено количество уникальных авторов и ID** - 1917 авторов и 1327 ID при помощи метода .nunique() - разница объясняется разным форматом написания имен одних и тех же авторов
8. **Приведение к единому формату написания авторов** - был создан датафрейм только с author и ID путем удаления столбцов. Далее были удалены дублирующися строки по столбцу ID (остались только уникальные ID, после чего - было проведено объединение с начальным датафреймом, в результате каждому уникальному ID сопоставился единственно возможный автор. В результате количество авторов стало равнятся 1322 при 1327 уникальных ID. При помощи Excel были обнаружены авторы с более чем одним ID. В датафрейм были внесены изменения при помощи .loc и итоговое количество уникальных авторов и ID сравнялось и  составило 1324 
9. **Формирование конечного датафрейма final для анализа данных** При помощи группировки по уникальным связкам ID и author и агреггирования суммы цитирований по столбцам был получен датафрейм final, содержащий в себе авторов, ID, данные по цитированию за 2020-2022 по отдельности и в сумме

##### 3. Добавление в датафрейм final информации по количеству **публикаций** в scopus авторов СПГУ за 2020-2022 по отдельности и в сумме
Оценка количества публикаций по годам. Применялся фильтр по каждому году, группировка с учетом фильтра, подсчет значений и объединение с финальным датафреймом. В результате в датафрейм добавлялись столбцы с количеством публикаций по каждому автору. Отдельно был создан столбец с суммой количества публикаций по годам.

##### 4. Добавление в датафрейм final информации по количеству **высокорейтинговых публикаций** авторов СПГУ за 2020-2022 по отдельности и в сумме 
1. **Создание датафрейма sjr объединением файлов scimagojr_2020(2021,2022).csv** содержащй информацию о всех источниках в scopus за 2020-2022 года и наивысших квартилях присвоенных этим источникам в каждом году по отдельности
2. **Объединение датафреймов scopus и sjr** - в результате, каждому источнику, в которых публиковались сотрудники СПГУ были присвоены наивысшие значения квартилей в 2020, 2021 и 2022 годах
3. **Исправление датафрейма scopus_sjr** - В датафрейме scopus - 4354 строки, в датафрейме scopus_sjr - 4361 строка - при объединении появились новые строки. Файлы скачаны для анализа в Excel и построчно через ЕСЛИ сравнены названия журналов Было выявлено дублирование, из-за названия журнала в файлах базы SJR - журнал Social Sciences в работе Автора Kornilova E.V. - данное дублирование не обнаруживалось через .duplicated. Журналу были присвоены значения Q2 и Q4 одновременно по каждому году во всех возможных сочетаниях. Видимо проблема заключалась в исходных файлах scimagojr_2020(2021,2022).csv, где одному названию журнала были присвоены одновременно и Q2 и Q4. Автор Kornilova E.V. публиковалась в журнале отнесенном в Q2, следовательно были проставлены Q2 в соответствующих ячейках и удалены дубликаты. Также были заменены значения квартилей "-" на "0" при помощи .replace
4. **Определение квартиля журнала в год опубликования статьи** Добавлен столбец quartile в котором находится квартиль по году написания статьи - по уточнению из email, при помощи лямбда функци, которая берет значение из столбца с годом опубликования и использует для получения столбца, содержащего квартиль для этого года и помещает в новый столбец - quartile
5. **Добавление в датафрейм final информации по количеству высокорейтинговых публикаций** Датафрейм scopus_sjr был сгруппирован по ID, author, year и quartile. Созданы отдельные столбцы для каждого года и категории. При группировке для каждого автора было подсчитано количество значений Q1 и Q2 в столбце квартиля и возвращена их сумма по годам. Создан отдельный столбец для общего количества высокорейтинговых публикаций за 3 года, путем суммирования значений по годам.

##### 5. Добавление в датафрейм final информации по количеству упоминаний Babyr и Tcvetkov авторами СПГУ 
1. **Постановка задачи - определение конкретных лиц**, чьи упоминания ищутся в ходе данной задачи. Учитывая что в исходных данных было несколько авторов с указанными фамилиями, было принято решение - найти количество упоминаний начальника управления по публикационной деятельности - Цветкова П.С. (Tcvetkov, P. - формат APA) также найти сколько раз упоминался начальник отдела наукометрического анализа - Бабырь Н.В. (Babyr, N. - формат APA)
2. **Создание столбцов в датафрейме scopus при помощи формулы и конструкции if else**, которая исчет указанных авторов в формате APA в цитированных работах авторов СПБПУ и возвращает количество упомянутых раз. Были заполнены два столбца датафрейма - mention_Tcvetkov и mention_Babyr
3. **Объединение датафрейма final и упоминаний** Проведено удаление лишних столбцов, объединение (left join) при помощи метода pd.merge по ключам ID и Автор. Переименование столбцов в необходимый и понятный вид, расположение столбцов с данными по публикациям по годам. 

##### 6. Cоздание таблицы для выполнения первого тестового задания **test_1.csv**
1. **Удаление лишних столбцов** датафрейма final с записью в датафрейм test_1
2. **Создания столбца "Цитирований за 2021-2022"** для задания, путем суммирования столбцов с цитированиями за 2021 и 2022 год
3. **Перемещение столбцов** в порядке, указанном в задании
4. **Сохранение датафрейма в таблицу test_1.csv** При помощи метода .to_csv 

## 3. Анализ публикационной активности сотрудников СПГУ
### Описание работы в файле 3.Excel.md и файлы работы test_2.xlsx, dashboard.jpg приложены в репозиторий, для анализа использовался Excel
#### Краткий алгоритм действий
##### 1. Определение показателей и наиболее подходящих визуальных представлений
1. Количество публикаций по годам - barplot
2. Среднее, медианное, мода количества публикаций - lineplot
3. Количество высокорейтинговых публикаций по годам - barplot
4. Среднее, медианное, мода количества высокорейтинговых публикаций - lineplot
5. Количество цитирований по годам - barplot
6. Среднее, медианное, мода количества высокорейтинговых публикаций - lineplot
7. Доля высокорейтинговых публикаций от общего количества статей по университету по годам - barplot
8. Топ 3 автора по количеству публикаций за 3 года - table
9. Топ 3 автора по количеству публикаций за последний год - table
10. Топ 5 авторов по количеству высокорейтинговых публикаций за 3 года  - table
11. Топ 3 автора по количеству высокорейтинговых публикаций за последний год - table
12. Топ 3 наиболее цитируемых автора  - table
###### Такой показатель как индекс хирша не рассматривался, поскольку мы имеем дело с анализом активности за последние три года и у нас в данных нет начальных значений индекса Хирша авторов
##### 2. Анализ по показателям и выводы
Использовались сводные таблицы, диаграммы и построение дашборда 
Выводы сделаны в ходе анализа дашборда
![image](https://github.com/fgomazov/test_spmi/assets/129670872/7ede3f5d-0233-4c29-b4f1-62ecc835ac5a)# Тестовое задание 
