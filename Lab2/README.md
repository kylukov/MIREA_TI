# README
aa-kylukov@yandex.ru

# Основы обработки данных с помощью R и Dplyr

aa-kylukov@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()

## Исходные данные

1.  Ноутбук с ОС MacOS
2.  RStudio
3.  Интерпретатор языка R 4.5.1

## Общий план выполнения работы

1.  Проанализировать встроенный в пакет dplyr набор данных starwars
2.  Выполнить аталитические задачи с помощью языка R

## Шаги

### 0. Атализ встроенного пакета dplyr инабора данных starwars

``` r
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
starwars
```

    # A tibble: 87 × 14
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Luke Sk…    172    77 blond      fair       blue            19   male  mascu…
     2 C-3PO       167    75 <NA>       gold       yellow         112   none  mascu…
     3 R2-D2        96    32 <NA>       white, bl… red             33   none  mascu…
     4 Darth V…    202   136 none       white      yellow          41.9 male  mascu…
     5 Leia Or…    150    49 brown      light      brown           19   fema… femin…
     6 Owen La…    178   120 brown, gr… light      blue            52   male  mascu…
     7 Beru Wh…    165    75 brown      light      blue            47   fema… femin…
     8 R5-D4        97    32 <NA>       white, red red             NA   none  mascu…
     9 Biggs D…    183    84 black      light      brown           24   male  mascu…
    10 Obi-Wan…    182    77 auburn, w… fair       blue-gray       57   male  mascu…
    # ℹ 77 more rows
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

### 1. Анализ количества строк в датафрейме

``` r
nrow(starwars)
```

    [1] 87

### 2. Анализ количества столбцов в датафрейме

``` r
ncol(starwars)
```

    [1] 14

### 3. Просмотр примерного вида датафрейма

``` r
glimpse(starwars)
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

### 4. Анализ уникальных рас персонажей

``` r
length(unique(starwars$species))
```

    [1] 38

### 5. Нахождение самого высокого персонажа

``` r
starwars %>% 
  arrange(desc(height)) %>% 
  slice(1)
```

    # A tibble: 1 × 14
      name      height  mass hair_color skin_color eye_color birth_year sex   gender
      <chr>      <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    1 Yarael P…    264    NA none       white      yellow            NA male  mascu…
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

### 6. Нахождеение всех персонажей ниже 170

``` r
starwars %>% 
  filter(height<170) %>%
  select(name, height, species)
```

    # A tibble: 22 × 3
       name                  height species       
       <chr>                  <int> <chr>         
     1 C-3PO                    167 Droid         
     2 R2-D2                     96 Droid         
     3 Leia Organa              150 Human         
     4 Beru Whitesun Lars       165 Human         
     5 R5-D4                     97 Droid         
     6 Yoda                      66 Yoda's species
     7 Mon Mothma               150 Human         
     8 Wicket Systri Warrick     88 Ewok          
     9 Nien Nunb                160 Sullustan     
    10 Watto                    137 Toydarian     
    # ℹ 12 more rows

### 7. Подсчет ИМТ для каждого персонажа

``` r
starwars %>% 
  mutate("IMT" = mass/(height*height)) %>% 
  select(name,IMT)
```

    # A tibble: 87 × 2
       name                   IMT
       <chr>                <dbl>
     1 Luke Skywalker     0.00260
     2 C-3PO              0.00269
     3 R2-D2              0.00347
     4 Darth Vader        0.00333
     5 Leia Organa        0.00218
     6 Owen Lars          0.00379
     7 Beru Whitesun Lars 0.00275
     8 R5-D4              0.00340
     9 Biggs Darklighter  0.00251
    10 Obi-Wan Kenobi     0.00232
    # ℹ 77 more rows

### 8. Нахождение 10 самых “вытянутых” персонажей

``` r
starwars %>% 
  mutate("MH" = mass/height) %>% 
  arrange(desc(MH)) %>%
  slice(1:10)
```

    # A tibble: 10 × 15
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Jabba D…    175  1358 <NA>       green-tan… orange         600   herm… mascu…
     2 Grievous    216   159 none       brown, wh… green, y…       NA   male  mascu…
     3 IG-88       200   140 none       metal      red             15   none  mascu…
     4 Owen La…    178   120 brown, gr… light      blue            52   male  mascu…
     5 Darth V…    202   136 none       white      yellow          41.9 male  mascu…
     6 Jek Ton…    180   110 brown      fair       blue            NA   <NA>  <NA>  
     7 Bossk       190   113 none       green      red             53   male  mascu…
     8 Tarfful     234   136 brown      brown      blue            NA   male  mascu…
     9 Dexter …    198   102 none       brown      yellow          NA   male  mascu…
    10 Chewbac…    228   112 brown      unknown    blue           200   male  mascu…
    # ℹ 6 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>, MH <dbl>

### 9. Нахождение среднего возраст персонажей каждой расы вселенной Звездных войн.

``` r
starwars %>% 
  group_by(species) %>% 
  filter(!is.na(birth_year)) %>% 
  summarise(avg = mean(birth_year)) %>% 
  select(species, avg)
```

    # A tibble: 15 × 2
       species          avg
       <chr>          <dbl>
     1 Cerean          92  
     2 Droid           53.3
     3 Ewok             8  
     4 Gungan          52  
     5 Human           53.7
     6 Hutt           600  
     7 Kel Dor         22  
     8 Mirialan        49  
     9 Mon Calamari    41  
    10 Rodian          44  
    11 Trandoshan      53  
    12 Twi'lek         48  
    13 Wookiee        200  
    14 Yoda's species 896  
    15 Zabrak          54  

### 10. Нахождение самого распространенного цвета глаз

``` r
starwars %>% 
  group_by(eye_color) %>%
  filter(!is.na(eye_color)) %>%
  summarise(k = n()) %>%
  arrange(desc(k)) %>%
  slice(1)
```

    # A tibble: 1 × 2
      eye_color     k
      <chr>     <int>
    1 brown        21

### 11. Подсчет средней длины имени в каждой расе

``` r
starwars %>% 
  group_by(species) %>%
  filter(!is.na(name)) %>%
  summarise(name_length = mean(nchar(name)))
```

    # A tibble: 38 × 2
       species   name_length
       <chr>           <dbl>
     1 Aleena          12   
     2 Besalisk        15   
     3 Cerean          12   
     4 Chagrian        10   
     5 Clawdite        10   
     6 Droid            4.83
     7 Dug              7   
     8 Ewok            21   
     9 Geonosian       17   
    10 Gungan          11.7 
    # ℹ 28 more rows

## Оценка результата

-   Получены практические навыки использования языка программирования R
    для обработки данных
-   Закреплены знания базовых типов данных языка R
-   Получены практические навыки использования функций обработки данных
    пакета dplyr

## Вывод

В ходе работы были успешно освоены базовые возможности языка R и
ключевые функции пакета dplyr для обработки данных. На примере датасета
starwars были выполнены типовые операции: фильтрация, сортировка,
группировка, добавление новых переменных и агрегация данных.
