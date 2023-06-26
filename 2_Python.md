# Анализ публикационной активности сотрудников СПГУ

## 1. Импортируем необходимые библиотеки
import pandas as pd

## 2.Загрузка и предобработка датафрейма spmi из таблицы scopus.csv
spmi  = pd.read_csv('/mnt/HC_Volume_18315164/home-jupyter/jupyter-f-gomazov/Pet/spmi/spmi_scopus.csv', encoding='windows-1251', sep=';')

### 2.1 Обработка данных в ячейках

#### В датафрейме spmi удаляем ; в конце ячеек в столбце IDs, чтобы не было лишних строк при разделении
`spmi['IDs'] = spmi['IDs'].str.rstrip(';')`
#### Проверяем на дубликаты и пропущенные значения
`print('Проверка на дубликаты')`

`print(spmi.duplicated().sum())`

`print('Проверка на пропущенные значения')`

`print(spmi.isnull().sum())`
#### Заполнение пропущенных значений нулями
`spmi = spmi.fillna(0)`

### 2.2 Разделение столбцов authors IDs affiliation по разделителям и удаление пробелов в начале
#### Разделение столбца authors на отдельные имена авторов
`spmi_authors = spmi.assign(authors=spmi['authors'].str.split(',')).explode('authors')`
`spmi_authors['authors'] = spmi_authors['authors'].str.strip()`
#### Разделение столбца IDs на отдельные идентификаторы авторов
`spmi_ids = spmi.assign(IDs=spmi['IDs'].str.split(';')).explode('IDs')`
`spmi_ids['IDs'] = spmi_ids['IDs'].str.strip()`
#### Разделение столбца affiliation на отдельные идентификаторы авторов
`spmi_affiliation = spmi.assign(affiliation=spmi['affiliation'].str.split(';')).explode('affiliation')
spmi_affiliation['affiliation'] = spmi_affiliation['affiliation'].str.strip()`
#### Проверка количества строк авторов
`print(spmi_authors.shape)`
`print(spmi_ids.shape)`
`print(spmi_affiliation.shape)`

###### Для выяснения причин несовпадения - были скачаны три таблицы и проанализированы в Excel 
`spmi_authors.to_csv('spmi_authors.csv')`

`spmi_ids.to_csv('spmi_ids.csv')`

`spmi_affiliation.to_csv('spmi_affiliation.csv')`
###### В данных таблицах было обнаружено расхождение одной статье  - Problems of Applying Pb-Free Technology in Soldering Electronic Components. Столбец affiliation разделенный по ; создал строку со следующий аффилиацией (без автора): 'Department of Electronic Systems, Saint Petersburg, Russian Federation'  

###### Изменения были внесены в исходный файл spmi_scopus.csv, поскольку ошибка только в одной статье и важно сохранить аффилиацию. Поскольку изменен исходный файл - перезапускаем ядро и проверяем, совпало ли количество значений (п.2.4)

###### Количество совпало - данный пункт перенесен в комментарии

### 2.3 Создание итоговой таблицы. 
#### Преобразование столбца IDs в список идентификаторов
`spmi_authors['ID'] = spmi_ids['IDs'].apply(lambda x: x.strip()).tolist()`
#### Преобразование столбца affiliation в список идентификаторов
`spmi_authors['aff'] = spmi_affiliation['affiliation'].apply(lambda x: x.strip()).tolist()`
#### Удаление столбца с IDs и affiliation
`spmi_authors = spmi_authors.drop('IDs', axis=1)`

`spmi_authors = spmi_authors.drop('affiliation', axis=1)`
#### Перемещение столбца ID после столбца Authors 
`id_col = spmi_authors.pop('ID')`

`spmi_authors.insert(1, 'ID', id_col)`
#### Перемещение столбца aff после столбца ID, ренейм столбца aff в affiliation, ренейм столбца authors в author
`id_column = spmi_authors.pop('aff')`

`spmi_authors.insert(2, 'aff', id_column)`

`spmi_authors = spmi_authors.rename(columns={'aff': 'affiliation'})`

`spmi_authors = spmi_authors.rename(columns={'authors': 'author'})`
#### Фильтрация таблицы по условию аффилиации с СПГУ - наличие слова Mining в аффилиации
`spmi_authors = spmi_authors[spmi_authors['affiliation'].str.contains('Mining')]`

`scopus = spmi_authors.reset_index()`
#### Фильтрация таблицы по условию аффилиации с СПГУ - наличие слова Petersburg в фильтрованной аффилиации (содержит Mining)  
`scopus_affiliation_spmi = scopus[scopus['affiliation'].str.contains('Petersburg')]`

`scopus = scopus_affiliation_spmi`
#### Проверка таблицы на дубликаты 
`scopus.duplicated().sum()`

### 2.4 Формирование таблицы scopus с данными сотрудников СПГУ 
#### Удаление дубликатов
`scopus = scopus.drop_duplicates()`
#### Удаление появившегося столбца индекс
`scopus = scopus.drop('index', axis=1)`
#### Удаление столбца с аффилиациями, поскольку уже отсортированы только сотрудники СПГУ
`scopus = scopus.drop('affiliation', axis=1)`
#### Определение количества уникальных авторов 
`print(scopus.author.nunique())`
#### Определение количества уникальных ID 
`print(scopus.ID.nunique())`

### 2.5 Определение количества уникальных авторов
#### Приведение фамилий авторов в столбце author к единому формату написания
#### Количество авторов больше количества уникальных ID, поскольку в столбце author разный формат написания ФИО
#### Необходимо привести фамилии в единый формат через соотнесение с уникальными ID, для этого
#### Создается таблица состоящая только из ID и Авторов 
`Author_ID_only = scopus.drop(columns=['title', 'year', 'source', 'references',  'citation_2020', 'citation_2021', 'citation_2022', 'citation_2020_2022'])`

`Author_ID_only = Author_ID_only.drop_duplicates(subset=['ID'])`

`Author_ID_only['author'] = Author_ID_only['author'].str.strip()`
#### Изменение таблицы scopus без фамилий авторов для объединения 
`scopus = scopus.drop(columns=['author'])`
#### Каждому ID присвоена фамилия автора в единственном формате написания из таблицы Author_ID_only
`scopus = pd.merge(scopus, Author_ID_only, how = 'outer',  on=['ID'])`
`scopus.drop_duplicates().reset_index()`
#### Перемещение столбца author после столбца ID 
`id_column_1 = scopus.pop('author')`
`scopus.insert(1, 'author', id_column_1)`

### 2.6 - Анализ количества уникальных авторов 
#### формируем датафрейм final для тестового задания, оставляем только нужные столбцы - ID, авторы, цитаты по годам
`final = scopus.drop(columns=['title', 'year', 'source', 'references'])`
#### Группируем по ID и автору, аггрегируя сумму цитирований для каждого автора
`final = final.groupby(['ID','author'], as_index = False).agg({'citation_2020': 'sum','citation_2021': 'sum','citation_2022': 'sum','citation_2020_2022': 'sum'})`
#### Проверяем равенство уникальных авторов и уникальных ID 
`print(final.author.nunique())`

`print(final.ID.nunique())`

### 2.7 Поиск уникальных авторов с несколькими ID - необходимо проверить являются они тезками, или это дубликаты и 
#### внести изменения в таблицу scopus и датафрейм scopus, данное действие происходит в csv файле spmi через excel для анализа
`check = final.groupby(['author'], as_index = False).agg({'ID': 'count'})`

`check.query('ID > 1')`

### 2.8 Анализ пересечений авторов и ID - внесение изменений в датафрейм scopus
###### 5 авторов соотносятся с 2 ID, ни один автор не соотносится с 3 и более ID, остальные авторы имеют 1 ID 
###### Сохраняем csv файл `scopus.to_csv('scopus_p_2_15.csv')`для анализа в Excel. При помощи фильтров в Excel - были отсортированы авторы по фамилии в файле scopus_p_2_15.csv и найдены следующие ошибки  

###### 1. Chernobay V.I.  - 58153589100  - выполняется переадресация на 57194587676, следовательно этот ID был заменен в таблице  
###### 1. Chernobay V.I.  - 57194587676  - основной ID этого автора

`scopus.loc[scopus['ID'] == "58153589100", 'ID'] = "57194587676"`
###### 2. Ivanov A.     - 57674854600 - Ivanov A.S.       - совпадение из-за одинаковых иницалов - изменения внесены  
###### 2. Ivanov A.     - 57194280216 - Ivanov A.V.       - совпадение из-за одинаковых иницалов - изменения внесены
`scopus.loc[scopus['ID'] == "57674854600", 'author'] = 'Ivanov A.S.'`

`scopus.loc[scopus['ID'] == "57194280216", 'author'] = 'Ivanov A.V.'`
###### 3. Kuznetsova E. - 57366705000 - Kuznetsova E.A.   - совпадение из-за одинаковых иницалов - изменения внесены
###### 3. Kuznetsova E. - 57212385681 - Kuznetsova E.Y.   - совпадение из-за одинаковых иницалов - изменения внесены
`scopus.loc[scopus['ID'] == "57366705000", 'author'] = 'Kuznetsova E.A.'`

`scopus.loc[scopus['ID'] == "57212385681", 'author'] = 'Kuznetsova E.Y.'`
###### 4. Lavrik A.     - 57201615009 - Lavrik A.Y.       - совпадение из-за одинаковых иницалов - изменения внесены  
###### 4. Lavrik A.     - 57573296300 - Lavrik Anna       - совпадение из-за одинаковых иницалов - изменения внесены
`scopus.loc[scopus['ID'] == "57201615009", 'author'] = 'Lavrik A.Y.'`

`scopus.loc[scopus['ID'] == "57573296300", 'author'] = 'Lavrik Anna'`
###### 5. Vasiliev D.A.  - 57223084202 - аффилицация с СПГУ
###### 5. Vasiliev D.A.  - 55867855600 - аффилиация не с СПГУ, удален
`scopus = scopus.drop(scopus[scopus['ID'] == "55867855600"].index)`
###### Таким образом только одному автору принадлежали 2 ID, а прочие ID - уникальны и принадлежат людям со схожими инициалами, Также обнаружен автор без аффилиации с горным университетом. В ходе этого пункта - формат фамилий брался из исходной таблицы задания scopus_citation_tracker

###### В исходной таблице scopus формат фамилий отличался, поэтому, были повторены предыдущие шаги. В результате чего - были обнаружены 10 авторов с 2 ID, 5 из которых указаны выше 
######  Остальные 5 авторов (формат указания: фамилия в таблице scopus - ID - фамилия в таблице Scopus_Citation_Tracker )
###### 1. Fedorov A.T.-7402999353-Fedorov A. и Fedorov A.T.-57210103208-Fedorov A.T. - одинаковые инициалы (scopus)
###### 2. Mikhailov A.V.-57196260711-Mikhailov A. и  Mikhailov A.V.-57218597367-Mikhailov A.V. - одинаковые инициалы (scopus)
###### 3. Vasilev B.Y.-55803115000-Vasilev B. и Vasilev B.Y.-58156604600-Vasilev B.Y. - одинаковые инициалы (scopus)
###### 4. Vasilyeva M.A.-57117630800- Vasileva M. и Vasilyeva M.A.-57224741212-Vasilyeva M.A.- одинаковые инициалы (scopus)
###### 5. Tcvetkov P.   - 57222546007 - выполняется переадресация на 57191636882   - основной ID этого автора - в таблице Scopus_Citation_Tracker он указан как Tsvetkov P.  Учитывая что правильное написание в профиле Scopus Tcvetkov P., то в итоговой таблице ID и фамилия были заменены
`scopus.loc[scopus['author'] == "Tsvetkov P.", 'author'] = 'Tcvetkov P.'`

`scopus.loc[scopus['ID'] == "57222546007", 'ID'] = "57191636882"`

### 2.9 Cоздание датафрейма final из измененного по результатам анализа датафрейма scopus 
`final = scopus.drop(columns=['title', 'year', 'source', 'references'])`

`final = final.groupby(['ID','author'], as_index = False).agg({'citation_2020': 'sum','citation_2021': 'sum','citation_2022': 'sum','citation_2020_2022': 'sum'})`

`final['author'] = final['author'].apply(lambda x: x.strip())`
#### Проверка равенства уникальных авторов и уникальных ID 
`print(final.author.nunique())`

`print(final.ID.nunique())`

### 2.10 Вывод: Сформирован датафрейм final, состоящий из уникальных 1324 авторов с аффилиацией в СПГУ и их ID
#### Часть таблицы из первого задания (Автор, ID) заполнена. Данные по их статьям, содержатся в датафрейме scopus с привязкой к уникальным связкам авторов и ID

## 3 - Добавление количества публикаций за 2020-2022 года по отдельности и вместе
### 3.1 Оценка количества публикаций за 2020 год
##### Фильтр по количеству публикаций в 2020 году 
`filter_2020 = scopus['year'] == 2020`
#### Группировка по автору и ID с подсчтеом количества публикаций за 2020 год 
`publications_2020 = scopus[filter_2020].groupby(['ID', 'author'])['title'].count().reset_index()`

`final = pd.merge(final, publications_2020[['ID', 'title']], on='ID', how='left')`

`final = final.rename(columns = {'title': 'scopus_2020'})`
#### Приведение данных по публикациям в scopus за 2020 год к формату int64 и заполнение нулями значений,  когда у автора не было публикаций за 2020 год
`final = final.astype({"scopus_2020": "Int64"})`

`final = final.fillna(0)`

`final = final.drop_duplicates(subset=['ID'])`
### 3.3 Оценка количества публикаций за 2021 год
#### Фильтр по количеству публикаций в 2021 году 
`filter_2021 = scopus['year'] == 2021`
#### Группировка по автору и ID с подсчтеом количества публикаций за 2021 год 
`publications_2021 = scopus[filter_2021].groupby(['ID', 'author'])['title'].count().reset_index()`

`final = pd.merge(final, publications_2021[['ID', 'title']], on='ID', how='left')`

`final = final.rename(columns = {'title': 'scopus_2021'})`
#### Приведение данных по публикациям в scopus за 2021 год к формату int64 и заполнение нулями значений,  когда у автора не было публикаций за 2021 год
`final = final.astype({"scopus_2021": "Int64"})`

`final = final.fillna(0)`
### 3.4 Оценка количества публикаций за 2022 год
#### Фильтр по количеству публикаций в 2022 году 
`filter_2022 = scopus['year'] == 2022`
#### Группировка по автору и ID с подсчтеом количества публикаций за 2022 год 
`publications_2022 = scopus[filter_2022].groupby(['ID', 'author'])['title'].count().reset_index()`

`final = pd.merge(final, publications_2022[['ID', 'title']], on='ID', how='left')`

`final = final.rename(columns = {'title': 'scopus_2022'})`
#### Приведение данных по публикациям в scopus за 2022 год к формату int64 и заполнение нулями значений,  когда у автора не было публикаций за 2022 год
`final = final.astype({"scopus_2022": "Int64"})`

`final = final.fillna(0)`
### 3.5 Оценка количества публикаций за 2020-2022 год
`final['scopus_2020_2022'] = final[['scopus_2020', 'scopus_2021','scopus_2022']].sum(axis=1)`
#### Приведение к Int64
`final = final.astype({"scopus_2020_2022": "Int64"})`
#### Проверка на отсутствующие значения
`print(final.isnull().sum())`
### 3.6 Вывод. Датафрейм final дополнен данными по количеству публикаций за 2022 год и суммарному количеству публикаций за 2020-2022

## 4.Добавление количества публикаций высокорейтинговых журнала Q1 и Q2 за 2020-2022 года по отдельности и вместе
### 4.1 Cоздание датафрейма sjr по источникам и их квартилям
#### Импорт данных из файлов базы SJR по годам и проверка на дубликаты и пропущенные значения
`sjr_2020 = pd.read_csv('/mnt/HC_Volume_18315164/home-jupyter/jupyter-f-gomazov/Pet/spmi_scopus/scimagojr_2020.csv', encoding='windows-1251', sep=';')`

`sjr_2021 = pd.read_csv('/mnt/HC_Volume_18315164/home-jupyter/jupyter-f-gomazov/Pet/spmi_scopus/scimagojr_2021.csv', encoding='windows-1251', sep=';')`

`sjr_2022 = pd.read_csv('/mnt/HC_Volume_18315164/home-jupyter/jupyter-f-gomazov/Pet/spmi_scopus/scimagojr_2022.csv', encoding='windows-1251', sep=';')`

#### Проверка данных на дубликаты
`print('Проверка на дубликаты')`

`print(sjr_2020.duplicated().sum())`

`print(sjr_2021.duplicated().sum())`

`print(sjr_2022.duplicated().sum())`
#### Удаление дубликатов
`sjr_2020 = sjr_2020.drop_duplicates()`

`sjr_2021 = sjr_2021.drop_duplicates()`

`sjr_2022 = sjr_2022.drop_duplicates()`
#### Проверка данных на пропущенные значения
`print('Проверка на пропущенные значения')`

`print(sjr_2020.isnull().sum())`

`print(sjr_2021.isnull().sum())`

`print(sjr_2022.isnull().sum())`
#### Внешнее объединение таблиц с SJR по годам, поскольку нужны данные обо всех источниках - получаем датафрейм sjr
`sjr_20_21 = pd.merge(sjr_2020, sjr_2021, how = 'outer',  on=['source', 'type'])`

`sjr = pd.merge(sjr_20_21, sjr_2022, how = 'outer',  on=['source', 'type'])`

### 4.2 Создание и анализ датафрейма scopus_sjr с публикациями сотрудников СПГУ и квартилями журналов 
###### Применен left join, поскольку, например, часть журналов перестала индексироваться в скопус, а материалы конференций могут не иметь квартиля
`scopus_sjr = pd.merge(scopus, sjr, how = 'left', on = ['source'])`
####  проверка дублиткатов в таблице 
`print('Проверка на дубликаты')`

`print(scopus_sjr.duplicated().sum())`
####  проверка пропущенных значения
`print('Проверка на пропущенные значения')`

`print(scopus_sjr.isnull().sum())`
#### заполнение пропущенных значений нулями
`scopus_sjr = scopus_sjr.fillna(0)`
#### Сравнение по количеству строк датафрейма scopus с датафреймом scopus_sjr
`print('Проверка на количество строк')`

`print(scopus.shape)`

`print(scopus_sjr.shape)`
###### В датафрейме scopus - 4354 строки, в датафрейме scopus_sjr - 4361 строка - при объединении появились новые строки. Файлы скачаны для анализа в Excel и построчно через ЕСЛИ сравнены названия журналов. Было выявлено дублирование, из-за названия журнала в файлах базы SJR - журнал Social Sciences в работе Автора Kornilova E.V. - данное дублирование не обнаруживалось через .duplicated 
#### Проверка типов значений в столбцах квартилей по годам
`print('Проверка на типы значений квартилей')`

`print(scopus_sjr.quartile_2020.unique())`

`print(scopus_sjr.quartile_2021.unique())`

`print(scopus_sjr.quartile_2022.unique())`

### 4.3 Исправление данных датафрейма scopus_sjr
#### Замена значений '-' на 0
`scopus_sjr['quartile_2020'] = scopus_sjr['quartile_2020'].replace('-', 0)`

`scopus_sjr['quartile_2021'] = scopus_sjr['quartile_2021'].replace('-', 0)`

`scopus_sjr['quartile_2022'] = scopus_sjr['quartile_2022'].replace('-', 0)`
#### Исключение дублирования с журналом Social Sciences
`mask = (scopus_sjr['source'] == 'Social Sciences')`
#### Заменяем данные строки на Q2(так как журнал в 2020-2022 находился в Q2) и убираем дубликаты, сбрасывая индекс
`scopus_sjr.loc[mask, ['quartile_2020', 'quartile_2021', 'quartile_2022']] = 'Q2'`

`scopus_sjr = scopus_sjr.drop_duplicates().reset_index()`

`scopus = scopus.drop_duplicates().reset_index()`

`print('Проверка на количество строк')`

`print(scopus.shape)`

`print(scopus_sjr.shape)`

### 4.4 Изменение данных в датафрейме scopus_sjr
#### Добавлен столбец quartile в котором находится квартиль по году написания статьи - по уточнению из email
`scopus_sjr['quartile'] = scopus_sjr.apply(lambda row: row[f'quartile_{row.year}'], axis=1)`
#### Удалены колонки с квартилями по годам из файла с источниками 
`scopus_sjr = scopus_sjr.drop('quartile_2020', axis=1)`
`scopus_sjr = scopus_sjr.drop('quartile_2021', axis=1)`
`scopus_sjr = scopus_sjr.drop('quartile_2022', axis=1)`

### 4.5 Дополнение таблицы final данными по квартилям
#### Группируем исходный датафрейм по ID и author, считаем количество публикаций в каждой категории
`grouped = scopus_sjr.groupby(['ID', 'author', 'year', 'quartile']).size().reset_index(name='count')`
#### Cоздаем отдельные столбцы для каждого года и категории
`grouped['quartile12_2020'] = grouped.apply(lambda row: 1 if row['year'] == 2020 and row['quartile'] in ['Q1', 'Q2'] else 0, axis=1)`

`grouped['quartile12_2021'] = grouped.apply(lambda row: 1 if row['year'] == 2021 and row['quartile'] in ['Q1', 'Q2'] else 0, axis=1)`

`grouped['quartile12_2022'] = grouped.apply(lambda row: 1 if row['year'] == 2022 and row['quartile'] in ['Q1', 'Q2'] else 0, axis=1)`
#### Группируем по ID и author, суммируем количество публикаций в каждой категории и каждом году
`grouped = grouped.groupby(['ID', 'author']).agg({'quartile12_2020': 'sum', 'quartile12_2021': 'sum', 'quartile12_2022': 'sum', 'count': 'sum'}).reset_index()`
#### Создаем столбец quartile12, который принимает значение 1, если хотя бы один из столбцов quartile12_2020, quartile12_2021, quartile12_2022 равен 1, иначе 0
`grouped['quartile12_2020_2022'] = grouped[['quartile12_2020','quartile12_2021','quartile12_2022']].sum(axis=1)`
#### Объединение датафрейма final с группированным датафреймом grouped по ID и author
`final['ID'] = final['ID'].astype('object')`

`grouped['ID'] = grouped['ID'].astype('object')`

`final['author'] = final['author'].str.strip()`

`grouped['author'] = grouped['author'].str.strip()`

`result = final.merge(grouped[['ID', 'author', 'quartile12_2020', 'quartile12_2021', 'quartile12_2022', 
'quartile12_2020_2022']], on=['ID', 'author'], how='left')`
#### Заполнение нулями отсутствующих значений
`result[['quartile12_2020', 'quartile12_2021', 'quartile12_2022', 'quartile12_2020_2022']] = result[['quartile12_2020', 'quartile12_2021', 'quartile12_2022', 'quartile12_2020_2022']].fillna(0).astype('int')`
#### Перезапись датафрейм final для тестового задания
`final = result`

### 4.6 Вывод - сформирована таблица final с данными для тестового задания, за исключением упоминания фамилий авторов Babyr и Tcvetkov

## 5. Добавление количества упоминаний авторов Babyr и Tcvetkov сотрудниками СПГУ
##### Количество упоминаний авторов Babyr и Tcvetkov в каждой статье из столбца пристатейных ссылок. Учитывая, что есть два автора с фамилией Babyr, и, возможно, более одного автора с фамилией Tcvetkov, то было принято решение - найти количество упоминаний начальника управления по публикационной деятельности - Цветкова П.С. и найти сколько раз упоминался начальник отдела наукометрического анализа - Бабырь Н.В.

### 5.1 Создание столбцов в датафрейме scopus
#### mention_Tcvetkov - сколько раз упомянут Цветков П.С. - в формуле использовано формат фамилии APA
`def count_mention_1(row):
    if isinstance(row['references'], str):
        return row['references'].count('Tcvetkov, P')
    else:
        return 0
scopus['mention_Tcvetkov'] = scopus.apply(count_mention_1, axis=1)`
#### mention_Babyr - сколько раз упомянут Бабырь Н.В. - в формуле использовано формат фамилии APA
`def count_mention(row):
    if isinstance(row['references'], str):
        return row['references'].count('Babyr, N.')
    else:
        return 0
scopus['mention_Babyr'] = scopus.apply(count_mention, axis=1)`

### 5.2 Дополнение таблицы final данными об упоминаниях
#### Создание датафрейма scopus_mention с количеством упоминаний для объединения
`scopus_mention = scopus`
#### Подготовка датафрейма к объединению с final - удаление лишних столбцов, удаление дубликатов, подсчет суммы упоминаний
`scopus_mention = scopus_mention.drop(columns = ['title','year', 'source', 'references','citation_2021', 'citation_2022'])`
`scopus_mention = scopus_mention.groupby(['ID', 'author']).agg({'mention_Tcvetkov': 'sum', 'mention_Babyr': 'sum'}).reset_index()`
`scopus_mention = scopus_mention.drop_duplicates()`
#### Объединение таблицы final и scopus_mention
`final = pd.merge(final, scopus_mention, how = 'left',  on=['ID', 'author'])`

`final[['mention_Tcvetkov', 'mention_Babyr']] = final[['mention_Tcvetkov', 'mention_Babyr']].fillna(0).astype('int')`

`final = final.rename(columns={'ID': 'Идентификатор Scopus', 'author': 'Автор', 'citation_2020': 'Цитирований за 2020', 'citation_2021': 'Цитирований за 2021', 'citation_2022': 'Цитирований за 2022','citation_2020_2022': 'Цитирований за 2020 - 2022', 'scopus_2020': 'Публикаций Scopus за 2020', 'scopus_2021': 'Публикаций Scopus за 2021', 'scopus_2022': 'Публикаций Scopus за 2022', 'scopus_2020_2022': 'Публикаций Scopus за 2020-2022', 'quartile12_2020': 'Высокорейтинговых публикаций за 2020', 'quartile12_2021': 'Высокорейтинговых публикаций за 2021', 'quartile12_2022': 'Высокорейтинговых публикаций за 2022', 'quartile12_2020_2022': 'Высокорейтинговых публикаций за 2020-2022', 'mention_Tcvetkov': 'Цветков П.С. был процитирован','mention_Babyr': 'Бабырь Н.В. был процитирован'})`

`final = final[['Идентификатор Scopus', 'Автор', 'Публикаций Scopus за 2020', 'Высокорейтинговых публикаций за 2020', 'Цитирований за 2020', 'Публикаций Scopus за 2021', 'Высокорейтинговых публикаций за 2021', 'Цитирований за 2021', 'Публикаций Scopus за 2022', 'Высокорейтинговых публикаций за 2022', 'Цитирований за 2022', 'Публикаций Scopus за 2020-2022', 'Высокорейтинговых публикаций за 2020-2022', 'Цитирований за 2020 - 2022', 'Цветков П.С. был процитирован','Бабырь Н.В. был процитирован']]`

### 5.3 Выгрузка финальной таблицы для анализа
`final.to_csv('final.csv')`

`final.head()`

### 6 Выгрузка таблицы для первого тестового задания
#### Удаление ненужных столбцов
`test_1 = final.drop(columns = ['Публикаций Scopus за 2020','Публикаций Scopus за 2021', 'Цитирований за 2020', 'Цитирований за 2021','Цитирований за 2022', 'Цитирований за 2020 - 2022', 'Высокорейтинговых публикаций за 2020', 'Высокорейтинговых публикаций за 2021', 'Высокорейтинговых публикаций за 2022'])`
#### Создание столбца с цитированием за 2021-2022 года
`test_1['Цитирований за 2021 - 2022'] = final['Цитирований за 2021'] + final['Цитирований за 2022']`
#### Приведение расположения столбцов в соотвествии с заданием
`test_1 = test_1[['Автор', 'Идентификатор Scopus',  'Публикаций Scopus за 2022', 'Публикаций Scopus за 2020-2022', 'Высокорейтинговых публикаций за 2020-2022', 'Цитирований за 2021 - 2022', 'Бабырь Н.В. был процитирован', 'Цветков П.С. был процитирован']]`
#### Сохранение файла с первым заданием
`test_1.to_csv('test_1.csv')`
