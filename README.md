# Анализ google merch store

#### Анализ факторов влияющих на средний чек

Данный анализ проводился в учебных целях.

#### ЦЕЛИ:
1. Разобраться как работает такой инструмент, как BigQuery; 
2. Научится делать запросы и выгружать данные из BigQuery;
3. Провести обработку данных;
4. Провести первичный анализ данных;
5. Сегментировать аудиторию;
6. Определить ЦА на основе данных и данных из GA (демо аккаунт);
7. Провести когортный анализ;
8. Построить модель данных, которая позвоилт определить факторы влияющие в покупку и средний чек;
9. Построить [дашборд](https://app.powerbi.com/view?r=eyJrIjoiYmU4YmYwOWQtNTc3MS00ZTAzLThkOWItNjVkNjExYmQxZTIyIiwidCI6IjZhNGRlZTAxLWMzZjUtNGQ0Yi1iZGQyLTllMWYxNDgyYWM1ZCIsImMiOjl9&pageName=ReportSection) в Power BI;
10. Определить факторы, влияющие на средний чек и выручку.

##### ИНСТРУМЕНТЫ:
1. Google BigQuery;
2. Google analytics;
3. MS Excel (Построениe многофакторной модели данных);
4. Python (Pandas, NumPy, matplotlib);
5. SQL (запросы в BigQuery);
6. MS Power BI.


#### Ход работы визуализировал с помощью UML схемы. 
Мне кажется она отлично может показывать ход работы, инструменты которые я использовал в проекте, какие шаги предпринимал и какие выводы сделал. (Дашборд в стиле БА/СА)  

<img width="1429" alt="Снимок экрана 2023-07-23 в 12 10 18" src="https://github.com/SmirnovArtemDA/google_mech_store/assets/139784954/c107e11e-34eb-47f7-8a60-db3596773552">


#### ЗАПРОС В BigQuery:

```
SELECT
   fullVisitorId,
   date,
   trafficSource.campaign,
   trafficSource.keyword,
   trafficSource.medium,
   trafficSource.source,
   device.browser,
   device.deviceCategory,
   geoNetwork.continent,
   geoNetwork.country,
   h.page.pagePath,
   h.item.transactionId,
   p.productRevenue,
   p.productQuantity,
   p.productPrice,
   p.v2ProductCategory,
   p.v2ProductName
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*` ,
   UNNEST(hits) as h,
   UNNEST(h.product) as p
WHERE  SAFE_CAST(_TABLE_SUFFIX as INT64) >= 20170101
   and SAFE_CAST(_TABLE_SUFFIX as INT64) < 20170501
```
К сожалению, далеко не все данные демоаккаунта открыты для использования. Например, я захотел посмотреть город пользователя (geoNetwork.city), то мне в выводе данных в каждой строке таблицы увидел запись о том, что данная информация недоступна. 

Для определения, какое действие было выполнено одним и тем же пользователем, буду пользоваться идентификатором fullVisitorId. Это некий обезличенный идентификатор, который, однако, позволит определить, что действия на сайте были совершены одним и тем же человеком (с некоторыми ограничениями, конечно).

Каждая строка таблицы — это отдельный сеанс, поэтому многие столбцы имеют вложенную структуру: например, в рамках одного сеанса пользователь мог посещать несколько страниц или покупать несколько товаров. Поэтому такие поля приходится «раскрывать» — придавать им плоский табличный формат. Я это сделал с помощью функции **UNNEST()**. 

