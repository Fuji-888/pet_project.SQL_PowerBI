#  Дашборд для анализа продаж в PowerBI -проект для портфолио аналитика данных 

![Иллюстрация к проекту](https://github.com/Fuji-888/pet_project_SQL_PBI/blob/main/Dashboard.png)

![Иллюстрация к проекту](https://github.com/Fuji-888/pet_project_SQL_PBI/blob/main/sql1.png)

![Иллюстрация к проекту](https://github.com/Fuji-888/pet_project_SQL_PBI/blob/main/sql2.png)

 
  Я представил себя на месте аналитика данных и сделал дашборд для отслеживания показателей, связанных с продажами.

## Таблица 

  В самом начале я составил таблицу с целями текущей работы, чем мой дашборд поможет бизнес-пользователю в его задачах. Далее я начертил компановку дашборда, подобрал метрики, определился с цветовой схемой. 

![Иллюстрация к проекту](https://github.com/Fuji-888/pet_project_SQL_PBI/blob/main/Анализ%20проекта%20относительно%20бизнес-пользователя%20и%20цели%20работы.png)

## Преобразование данных с помощью SQL
  Далее из базы данных (общедоступная AdventureWorks) с помощью SQL я извлек необходимые мне таблицы и преобразовал их. Сохранил все таблицы в формате csv. Код с комментариями. 

## FACT_InternetSales

```sql
SELECT 
  [ProductKey], 
  [OrderDateKey], 
  [DueDateKey], 
  [ShipDateKey], 
  [CustomerKey], 
  [SalesOrderNumber], 
  [SalesAmount] 
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE 
  LEFT (OrderDateKey, 4) >= 2019 -- Мог бы сделать YEAR(GETDATE())-2, чтобы показывались значения за 2 последних года, как написано в ТЗ (если база будет обновляться - то лучше сделать так), но база данных только до 2021 года, поэтому ставлю 2019 год
ORDER BY
  OrderDateKey ASC
```

## DIM_Products

```sql
SELECT 
  p.ProductKey, 
  p.ProductAlternateKey, 
  p.EnglishProductName AS [Product Name], 
  ps.EnglishProductSubcategoryName AS [Sub Category], --Подкатегория продукта из присоединенной таблицы [dbo].[DimProductSubcategory]
  pc.EnglishProductCategoryName AS [Product Category], -- Категория продукта из присоединенной таблицы [dbo].[DimProductSubcategory]
  p.Color AS [Product Color], 
  p.Size AS [Product Size], 
  p.ProductLine AS [Product Line], 
  p.ModelName AS [Product Model Name], 
  p.EnglishDescription AS [Product Description], 
  ISNULL (p.Status, 'Outdated') AS [Product Status] -- Заменяем все NULL значения на 'Outdated'
FROM 
  [AdventureWorksDW2019].[dbo].[DimProduct] as p
  LEFT JOIN dbo.DimProductSubcategory AS ps ON ps.ProductSubcategoryKey = p.ProductSubcategoryKey 
  LEFT JOIN dbo.DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey -- Присоеднияем 2 таблицы, чтобы показать больше информации о продукте (Категорию и Подкатегорию)
ORDER BY
  p.ProductKey ASC
```

## DIM_Customers

```sql
SELECT 
  c.CustomerKey, 
  c.FirstName AS [First Name], 
  c.LastName AS [Last Name], 
  c.FirstName + ' ' + c.LastName AS "Full Name", -- Объединил имя и фамилию в полное имя
  CASE 
	WHEN c.Gender = 'M' THEN 'Male' 
	WHEN c.Gender = 'F' THEN 'Female' 
	END AS Gender, 
  c.DateFirstPurchase, 
  g.city AS [Customer City] -- Вывожу столбец с названием города из присоединенной таблицы
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] AS c 
  LEFT JOIN [dbo].[DimGeography] AS g ON c.GeographyKey = g.GeographyKey -- Выбираю LEFT JOIN,а не INNER JOIN (хотя в данном случае кол-во строк будет одинаковым при обоих командах), потому что нам нужны все клиенты, а если ВДРУГ в таблице измерений (dbo.DimGeography) не окажется ключа географии - то клиент отсеится, что нам не надо 
ORDER BY 
  CustomerKey ASC -- Сортирую по возрастанию Ключа Клиента
```

## DIM_Calendar

```sql
SELECT 
  [DateKey], 
  [FullDateAlternateKey] AS Date, 
  [EnglishDayNameOfWeek] AS Day, 
  [EnglishMonthName] AS Month,
  LEFT([EnglishMonthName],3) AS MonthShort,
  [MonthNumberOfYear] AS MonthNumber, 
  [CalendarQuarter] AS Quarter, 
  [CalendarYear] AS Year 
FROM 
  [AdventureWorksDW2019].[dbo].[DimDate] 
WHERE 
  CalendarYear >= 2019
```

## Преобразование данных в PowerBI и модель данных

  Далее открыл все таблицы в PowerBI и преобразовал некоторые столбцы к нужному мне виду. 
После соединения ключей таблиц получилась модель данных, которая состоит из таблицы фактов и нескольких таблиц измерений. Тип модели данных - "звезда".

![Иллюстрация к проекту](https://github.com/Fuji-888/pet_project_SQL_PBI/blob/main/Data%20Model.png)

## Итогоый Дашборд

Дашборд интерактивен. Можно фильтровать данные с помощью срезов, а также выбирать нужные нам параметры, относительно которых будет выводиться информация.
  Дашборд включает в себя информацию о продажах, бюджете, количестве заказов. На нем присутствуют диаграмма с областями, показывающая выручку относитльено бюджета; график и гистограмма с группировкой, показывающий заказы и доход по месяцам; матрицы с информацией о продуктах и о клиентах; диаграмма дерево, показывающая распределение заказов по категориям.

  ![Иллюстрация к проекту](https://github.com/Fuji-888/pet_project_SQL_PBI/blob/main/Dashboard.png)
