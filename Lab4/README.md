# README
aa-kylukov@yandex.ru

# Исследование метаданных DNS трафика

aa-kylukov@yandex.ru

## Цель работы

1.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  Ноутбук с ОС MacOS
2.  RStudio
3.  Интерпретатор языка R 4.5.1
4.  Доступ в интернет

## Общий план выполнения

1.  Импортирт данных DNS – https:/
    /storage.yandexcloud.net/dataset.ctfsec/dns.zip
2.  Добавьте пропущенные данные о структуре данных (назначении столбцов)
3.  Преобразуйте данные в столбцах в нужный формат
4.  Просмотрите общую структуру данных с помощью функции glimpse()
5.  Сколько участников информационного обмена в сети Доброй Организации?
6.  Какое соотношение участников обмена внутри сети и участников
    обращений к внешним ресурсам?
7.  Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность.
8.  Найдите топ-10 доменов, к которым обращаются пользователи сети и
    соответственное количество обращений
9.  Опеределите базовые статистические характеристики (функция summary()
    ) интервала времени между последовательными обращениями к топ-10
    доменам.
10. Часто вредоносное программное обеспечение использует DNS канал в
    качестве канала управления, периодически отправляя запросы на
    подконтрольный злоумышленникам DNS сервер. По периодическим запросам
    на один и тот же домен можно выявить скрытый DNS канал. Есть ли
    такие IP адреса в исследуемом датасете?

### Шаг 1-3. Импорт данных и добавление названия столбцов

``` r
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
library(readr)

my_data <- read_tsv(
  file = "dns.log",
  col_names = FALSE,  # нет заголовков в файле
  comment = "#",      # пропускать строки с #
  show_col_types = FALSE
)

column_names <- c("ts", "uid", "id.orig_h", "id.orig_p", "id.resp_h", "id.resp_p", 
                  "proto", "trans_id", "query", "qclass", "qclass_name", "qtype", 
                  "qtype_name", "rcode", "rcode_name", "AA", "TC", "RD", "RA", 
                  "Z", "answers", "TTLs", "rejected")

colnames(my_data) <- column_names
head(my_data)
```

    # A tibble: 6 × 23
               ts uid   id.orig_h id.orig_p id.resp_h id.resp_p proto trans_id query
            <dbl> <chr> <chr>         <dbl> <chr>         <dbl> <chr>    <dbl> <chr>
    1 1331901006. CWGt… 192.168.…     45658 192.168.…       137 udp      33008 "*\\…
    2 1331901015. C36a… 192.168.…       137 192.168.…       137 udp      57402 "HPE…
    3 1331901016. C36a… 192.168.…       137 192.168.…       137 udp      57402 "HPE…
    4 1331901017. C36a… 192.168.…       137 192.168.…       137 udp      57402 "HPE…
    5 1331901006. C36a… 192.168.…       137 192.168.…       137 udp      57398 "WPA…
    6 1331901007. C36a… 192.168.…       137 192.168.…       137 udp      57398 "WPA…
    # ℹ 14 more variables: qclass <chr>, qclass_name <chr>, qtype <chr>,
    #   qtype_name <chr>, rcode <chr>, rcode_name <chr>, AA <lgl>, TC <lgl>,
    #   RD <lgl>, RA <lgl>, Z <dbl>, answers <chr>, TTLs <chr>, rejected <lgl>

### Шаг 4.1. Просмотр общей структуры данных с помощью glimpse()

``` r
glimpse(my_data)
```

    Rows: 427,935
    Columns: 23
    $ ts          <dbl> 1331901006, 1331901015, 1331901016, 1331901017, 1331901006…
    $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
    $ id.orig_h   <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
    $ id.orig_p   <dbl> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ id.resp_h   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
    $ id.resp_p   <dbl> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
    $ trans_id    <dbl> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62…
    $ query       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass      <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
    $ qclass_name <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
    $ qtype       <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "32"…
    $ qtype_name  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
    $ rcode       <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rcode_name  <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
    $ AA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ TC          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ RD          <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRU…
    $ RA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ Z           <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
    $ answers     <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ TTLs        <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rejected    <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…

### Шаг 4.2. Просмотр количества участников информационного обмена всети Доброй Организации

``` r
unique_source_ips <- unique(my_data$id.orig_h)
unique_destination_ips <- unique(my_data$id.resp_h)

# Объединяем все уникальные IP-адреса
all_ips <- unique(c(unique_source_ips, unique_destination_ips))

# Выводим результат
cat("Количество участников информационного обмена:", length(all_ips), "\n")
```

    Количество участников информационного обмена: 1359 

### Шаг 5. Отношение участников обмена внутри сети и участников обращений к внешним ресурсам

``` r
internal_sources <- sum(grepl("^(10\\.|192\\.168\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.)", all_ips))
external_sources <- length(all_ips) - internal_sources
internal_sources / external_sources
```

    [1] 13.77174

### Шаг 6. Найдите топ-10 участников сети, проявляющих наибольшую сетевую активность.

``` r
my_data %>%
  group_by(id.orig_h)%>%
  count(sort = TRUE) %>%
  as_tibble() %>%
  head(10)
```

    # A tibble: 10 × 2
       id.orig_h           n
       <chr>           <int>
     1 10.10.117.210   75943
     2 192.168.202.93  26522
     3 192.168.202.103 18121
     4 192.168.202.76  16978
     5 192.168.202.97  16176
     6 192.168.202.141 14967
     7 10.10.117.209   14222
     8 192.168.202.110 13372
     9 192.168.203.63  12148
    10 192.168.202.106 10784

### Шаг 7. Найдите топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

``` r
my_data %>%
  group_by(query)%>%
  count(sort = TRUE) %>%
  as_tibble() %>%
  head(10)
```

    # A tibble: 10 × 2
       query                                                                       n
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x… 10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

### Шаг 8. Опеределите базовые статистические характеристики (функция summary() ) интервала времени между последовательными обращениями к топ-10 доменам.

``` r
library(dplyr)

time_analysis <- my_data %>%
  count(query, name = "total_requests") %>%
  slice_max(total_requests, n = 10) %>%
  inner_join(my_data, by = "query") %>%
  group_by(query) %>%
  arrange(ts, .by_group = TRUE) %>%
  summarise(
    requests = n(),
    intervals = list(diff(sort(ts))),
    .groups = 'drop'
  ) %>%
  # Вычисляем статистики
  mutate(
    min_int = sapply(intervals, min),
    q1_int = sapply(intervals, quantile, 0.25),
    median_int = sapply(intervals, median),
    mean_int = sapply(intervals, mean),
    q3_int = sapply(intervals, quantile, 0.75),
    max_int = sapply(intervals, max),
    sd_int = sapply(intervals, sd)
  ) %>%
  select(-intervals) %>%
  arrange(median_int)

print(time_analysis)
```

    # A tibble: 10 × 9
       query       requests min_int q1_int median_int mean_int q3_int max_int sd_int
       <chr>          <int>   <dbl>  <dbl>      <dbl>    <dbl>  <dbl>   <dbl>  <dbl>
     1 "teredo.ip…    39273       0  0          0         2.94  0.510  50388.   256.
     2 "tools.goo…    14057       0  0          0         8.19  1      50365.   428.
     3 "*\\x00\\x…    10401       0  0.150      0.5      11.2   1.5    52724.   521.
     4 "HPE8AA67"      6929       0  0.75       0.75     16.6  25.5    50044.   602.
     5 "WPAD"          9134       0  0.75       0.75     12.6   1.11   50049.   524.
     6 "ISATAP"        6569       0  0.75       0.760    17.5   1.05   51998.   656.
     7 "safebrows…    11658       0  0          1        10.0   2.01   49952.   464.
     8 "www.apple…    13390       0  0          1         8.61  3.01   50964.   443.
     9 "time.appl…    13109       0  0.370      1.76      8.67  4.72   50924.   446.
    10 "44.206.16…     7248       0  2.09       4        16.0  20.1    49680.   584.

### Шаг 9. Поиск IP-адресов с периодическими запросами на один домен (из топ-10 доменов)

``` r
library(dplyr)

# Находим IP-адреса с периодическими запросами к топ-10 доменам
periodic_ips <- my_data %>%
  # Фильтруем только топ-10 доменов
  semi_join(
    count(., query) %>% slice_max(n, n = 10),
    by = "query"
  ) %>%
  # Группируем по IP и домену
  group_by(id.orig_h, query) %>%
  filter(n() >= 5) %>%  # Минимум 5 запросов для анализа периодичности
  arrange(ts) %>%
  summarise(
    request_count = n(),
    time_intervals = list(diff(ts)),
    # Анализ периодичности
    mean_interval = mean(diff(ts)),
    sd_interval = sd(diff(ts)),
    cv = sd_interval / mean_interval,  # Коэффициент вариации
    # Статистика регулярности
    min_interval = min(diff(ts)),
    max_interval = max(diff(ts)),
    range_ratio = max_interval / min_interval,
    .groups = 'drop'
  ) %>%
  # Фильтруем по признакам периодичности
  filter(
    cv < 0.5,           # Низкий коэффициент вариации = высокая регулярность
    request_count >= 10, # Достаточно запросов для анализа
    range_ratio < 10     # Относительно стабильный интервал
  ) %>%
  arrange(cv, request_count)  # Сортируем по регулярности

cat("IP-адреса с периодическими запросами на топ-10 доменов:\n")
```

    IP-адреса с периодическими запросами на топ-10 доменов:

``` r
print(periodic_ips)
```

    # A tibble: 3 × 10
      id.orig_h   query request_count time_intervals mean_interval sd_interval    cv
      <chr>       <chr>         <int> <list>                 <dbl>       <dbl> <dbl>
    1 192.168.20… ISAT…            90 <dbl [89]>             0.767       0.121 0.158
    2 192.168.20… ISAT…            33 <dbl [32]>             0.862       0.147 0.171
    3 192.168.0.3 ISAT…           108 <dbl [107]>            0.874       0.313 0.358
    # ℹ 3 more variables: min_interval <dbl>, max_interval <dbl>, range_ratio <dbl>

``` r
# Детальный анализ подозрительных IP
if(nrow(periodic_ips) > 0) {
  cat("\nДЕТАЛЬНЫЙ АНАЛИЗ ПОДОЗРИТЕЛЬНЫХ IP:\n")
  
  for(i in 1:nrow(periodic_ips)) {
    ip <- periodic_ips$id.orig_h[i]
    domain <- periodic_ips$query[i]
    
    cat(sprintf("\n%d. IP: %s -> Домен: %s\n", i, ip, domain))
    cat(sprintf("   Запросов: %d, Средний интервал: %.1f сек, CV: %.3f\n",
                periodic_ips$request_count[i],
                periodic_ips$mean_interval[i],
                periodic_ips$cv[i]))
    
  }
} else {
  cat("Периодические запросы не обнаружены.\n")
}
```


    ДЕТАЛЬНЫЙ АНАЛИЗ ПОДОЗРИТЕЛЬНЫХ IP:

    1. IP: 192.168.202.49 -> Домен: ISATAP
       Запросов: 90, Средний интервал: 0.8 сек, CV: 0.158

    2. IP: 192.168.202.147 -> Домен: ISATAP
       Запросов: 33, Средний интервал: 0.9 сек, CV: 0.171

    3. IP: 192.168.0.3 -> Домен: ISATAP
       Запросов: 108, Средний интервал: 0.9 сек, CV: 0.358

## Оценка результата

    •   Освоены базовые возможности R и пакета dplyr для работы с данными.
    •   Закреплены навыки фильтрации, сортировки, группировки и агрегации данных.
    •   Получен практический опыт анализа сетевого DNS-трафика.

## Вывод

Работа показала, как с помощью R и dplyr можно эффективно обрабатывать и
исследовать данные DNS-трафика, выявлять активных участников сети и
потенциально подозрительные IP-адреса.
