---
title: "ENUNCIADO taller en grupo Mat3 GIN 2025-2026"
author: "Taller"
lang: es
format:
  html:
    theme: superhero
    toc: true
    toc_depth: 4
    html-math-method: katex
    code-tools: true
    code-fold: true
    collapse: true
    keep-md: true
    code-overflow: wrap
---



# Instrucciones para el taller

Se entrega en grupos que deben de estar constituidos en la actividad de grupos. Los grupos son de 2 o 3 ESTUDIANTES, loa caso especiales consultadlos con el profesor para que los autorice.

**Enlaces y Bibliografía**

-   [R for data science, Hadley Wickham, Garret Grolemund.](https://r4ds.had.co.nz/)
-   [Fundamentos de ciencia de datos con R.](https://cdr-book.github.io/)
-   [Tablas avanzadas: kable, KableExtra.](https://haozhu233.github.io/kableExtra/awesome_table_in_html.html)
-   [Geocomputation with R, Robin Lovelace, Jakub Nowosad, Jannes Muenchow](https://r.geocompx.org/)
-   Apuntes de R-basico y tidyverse moodel MAT3.

## Objetivo MALLORCA

Leeremos los siguientes datos de la zona de etiqueta `mallorca` con el código siguiente:


::: {.cell}

```{.r .cell-code}
# Carga del archivo común pre-procesado
load("clean_data/mallorca/listing_common0.RData")

# Selección de variables de interés
listings0 = listings_common0 %>%
  select(id, scrape_id, listing_url,
         neighbourhood_cleansed, price,
         number_of_reviews,
         review_scores_rating,
         review_scores_cleanliness,
         review_scores_location,
         review_scores_value,
         accommodates,
         bathrooms_text,
         bedrooms,
         beds,
         minimum_nights,
         description,
         latitude,
         longitude,
         property_type,
         room_type)
```
:::


**listings**

Hemos cargado el objeto `listings0` que contiene los datos varios periodos de apartamentos de inside Airbnb de Mallorca seleccionando cuantas variables nos parecen más interesantes.

Separaremos la fecha del scrapping que es en la que se observó el fichero.


::: {.cell}

```{.r .cell-code}
listings0 = listings0 %>% 
  mutate(date = as.Date(substr(as.character(scrape_id), 1, 8), format="%Y%m%d"),
         .after = id)
```
:::


Ahora veamos la fechas de los scrapings y el número de veces que aparecen cada apartamentos.


::: {.cell}

```{.r .cell-code}
table(listings0$date)
```

::: {.cell-output .cell-output-stdout}

```

2023-12-17 2024-03-23 2024-06-19 2024-09-13 2024-12-14 2025-03-07 2025-06-15 
      9197       9197       9197       9197       9197       9197       9197 
2025-09-21 
      9197 
```


:::
:::


Vemos que cada apartamento aparece 8 veces una por periodo.


::: {.cell}

```{.r .cell-code}
table(table(listings0$id))
```

::: {.cell-output .cell-output-stdout}

```

   8 
9197 
```


:::
:::


Notemos que cada apartamento:

-   Queda identificado por id y por date que nos da el periodo en la que apareció el dato.
-   Las muestras disponibles son: 2023-12-17, 2024-03-23, 2024-06-19, 2024-09-13, 2024-12-14, 2025-03-07, 2025-06-15, 2025-09-21.

**reviews**

Estos datos necesitan leerse de forma adecuada.
**Nota:** Cargamos el archivo desde la carpeta `2024-09-13`.


::: {.cell}

```{.r .cell-code}
# Ruta ajustada a tus archivos
reviews = read_csv("data/mallorca/2024-09-13/reviews.csv.gz")
str(reviews)
```

::: {.cell-output .cell-output-stdout}

```
spc_tbl_ [387,059 × 6] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
 $ listing_id   : num [1:387059] 69998 69998 69998 69998 69998 ...
 $ id           : num [1:387059] 881474 4007103 4170371 4408459 4485779 ...
 $ date         : Date[1:387059], format: "2012-01-24" "2013-04-02" ...
 $ reviewer_id  : num [1:387059] 1595616 3868130 5730759 5921885 810469 ...
 $ reviewer_name: chr [1:387059] "Jean-Pierre" "Jo And Mike" "Elizabeth" "Jone" ...
 $ comments     : chr [1:387059] "This place was charming! Lorenzo himself is a very warm and engaging host and made us feel very welcome. \r<br/"| __truncated__ "We had a four night stay at this gorgeous apartment and it was absolutely perfect. It's really pretty, beautifu"| __truncated__ "Lor's apartment looks exactly like the pictures! It is perfectly located for historic Palma - close to the Cath"| __truncated__ "Wonderful place! 10/10. Charming, spacious and comfortable. Looks even more splendid than in the pictures. The "| __truncated__ ...
 - attr(*, "spec")=
  .. cols(
  ..   listing_id = col_double(),
  ..   id = col_double(),
  ..   date = col_date(format = ""),
  ..   reviewer_id = col_double(),
  ..   reviewer_name = col_character(),
  ..   comments = col_character()
  .. )
 - attr(*, "problems")=<externalptr> 
```


:::

```{.r .cell-code}
head(reviews)
```

::: {.cell-output .cell-output-stdout}

```
# A tibble: 6 × 6
  listing_id      id date       reviewer_id reviewer_name comments              
       <dbl>   <dbl> <date>           <dbl> <chr>         <chr>                 
1      69998  881474 2012-01-24     1595616 Jean-Pierre   "This place was charm…
2      69998 4007103 2013-04-02     3868130 Jo And Mike   "We had a four night …
3      69998 4170371 2013-04-15     5730759 Elizabeth     "Lor's apartment look…
4      69998 4408459 2013-05-03     5921885 Jone          "Wonderful place! 10/…
5      69998 4485779 2013-05-07      810469 Andrea        "My boyfriend and I, …
6      69998 4619699 2013-05-15     3318059 Devii         "We had a very last m…
```


:::
:::


**neighbourhoods.csv**

Son dos columnas y la primera es una agrupación de municipios (están NA) y la segunda es el nombre del municipio.


::: {.cell}

```{.r .cell-code}
# Ruta ajustada a tus archivos
municipios = read_csv("data/mallorca/2024-09-13/neighbourhoods.csv")
str(municipios)
```

::: {.cell-output .cell-output-stdout}

```
spc_tbl_ [53 × 2] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
 $ neighbourhood_group: logi [1:53] NA NA NA NA NA NA ...
 $ neighbourhood      : chr [1:53] "Alaró" "Alcúdia" "Algaida" "Andratx" ...
 - attr(*, "spec")=
  .. cols(
  ..   neighbourhood_group = col_logical(),
  ..   neighbourhood = col_character()
  .. )
 - attr(*, "problems")=<externalptr> 
```


:::

```{.r .cell-code}
head(municipios)
```

::: {.cell-output .cell-output-stdout}

```
# A tibble: 6 × 2
  neighbourhood_group neighbourhood
  <lgl>               <chr>        
1 NA                  Alaró        
2 NA                  Alcúdia      
3 NA                  Algaida      
4 NA                  Andratx      
5 NA                  Ariany       
6 NA                  Artà         
```


:::
:::


**neighbourhoods.geojson**

Es el mapa de Mallorca.


::: {.cell}

```{.r .cell-code}
# Leer el archivo GeoJSON (Ruta ajustada)
geojson_sf <- sf::st_read("data/mallorca/2024-09-13/neighbourhoods.geojson")
```

::: {.cell-output .cell-output-stdout}

```
Reading layer `neighbourhoods' from data source 
  `C:\Users\Alberto A G\Desktop\R proyectos\tallerMat3_AA.MS.AL.MA\data\mallorca\2024-09-13\neighbourhoods.geojson' 
  using driver `GeoJSON'
Simple feature collection with 53 features and 2 fields
Geometry type: MULTIPOLYGON
Dimension:     XY
Bounding box:  xmin: 2.303195 ymin: 39.26403 xmax: 3.479028 ymax: 39.96236
Geodetic CRS:  WGS 84
```


:::

```{.r .cell-code}
# Crear un mapa interactivo/estático
tmap_mode("plot") 
tm_shape(geojson_sf) +
  tm_polygons(col = "cyan", alpha = 0.6) +
  tm_layout(title = "Mapa - GeoJSON Mallorca con municipios")
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-7-1.png){width=672}
:::
:::


Tenéis que consultar en la documentación de inside Airbnb para saber que significa cada variable.

Responder las siguientes preguntas con formato Rmarkdown (.Rmd) o quarto (.qmd) y entregad la fuente un fichero en formato html como salida del informe.

## Pregunta 1 (**1punto**)

Del fichero con los datos de listings `listings0` calcula los estadísticos descriptivos de las variable `price` y de la variable `number_of_reviews` agrupados por municipio y por periodo.

Presenta los resultados con una tabla de kableExtra.


::: {.cell}

```{.r .cell-code}
# Cálculo de estadísticos descriptivos
tabla_resumen <- listings0 %>%
  group_by(neighbourhood_cleansed, date) %>%
  summarise(
    media_precio = mean(price, na.rm = TRUE),
    sd_precio = sd(price, na.rm = TRUE),
    media_reviews = mean(number_of_reviews, na.rm = TRUE),
    sd_reviews = sd(number_of_reviews, na.rm = TRUE),
    n = n(),
    .groups = "drop"
  )

# Visualización con KableExtra
tabla_resumen %>%
  kbl(caption = "Estadísticos descriptivos por Municipio y Periodo", digits = 2) %>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed")) %>%
  scroll_box(width = "100%", height = "400px")
```

::: {.cell-output-display}
`````{=html}
<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:400px; overflow-x: scroll; width:100%; "><table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
<caption>Estadísticos descriptivos por Municipio y Periodo</caption>
 <thead>
  <tr>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> neighbourhood_cleansed </th>
   <th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;"> date </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> media_precio </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> sd_precio </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> media_reviews </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> sd_reviews </th>
   <th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 425.23 </td>
   <td style="text-align:right;"> 884.07 </td>
   <td style="text-align:right;"> 39.11 </td>
   <td style="text-align:right;"> 65.62 </td>
   <td style="text-align:right;"> 62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 400.15 </td>
   <td style="text-align:right;"> 771.30 </td>
   <td style="text-align:right;"> 39.95 </td>
   <td style="text-align:right;"> 68.46 </td>
   <td style="text-align:right;"> 62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 439.57 </td>
   <td style="text-align:right;"> 831.16 </td>
   <td style="text-align:right;"> 41.92 </td>
   <td style="text-align:right;"> 71.49 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 599.92 </td>
   <td style="text-align:right;"> 1496.86 </td>
   <td style="text-align:right;"> 46.32 </td>
   <td style="text-align:right;"> 75.31 </td>
   <td style="text-align:right;"> 62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 567.16 </td>
   <td style="text-align:right;"> 1487.23 </td>
   <td style="text-align:right;"> 48.82 </td>
   <td style="text-align:right;"> 78.39 </td>
   <td style="text-align:right;"> 62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 537.38 </td>
   <td style="text-align:right;"> 1321.93 </td>
   <td style="text-align:right;"> 49.56 </td>
   <td style="text-align:right;"> 80.49 </td>
   <td style="text-align:right;"> 62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 724.13 </td>
   <td style="text-align:right;"> 1795.41 </td>
   <td style="text-align:right;"> 52.89 </td>
   <td style="text-align:right;"> 83.90 </td>
   <td style="text-align:right;"> 62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alaró </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 851.35 </td>
   <td style="text-align:right;"> 2020.01 </td>
   <td style="text-align:right;"> 56.24 </td>
   <td style="text-align:right;"> 85.96 </td>
   <td style="text-align:right;"> 62 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 210.38 </td>
   <td style="text-align:right;"> 164.43 </td>
   <td style="text-align:right;"> 20.68 </td>
   <td style="text-align:right;"> 33.54 </td>
   <td style="text-align:right;"> 956 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 209.82 </td>
   <td style="text-align:right;"> 170.98 </td>
   <td style="text-align:right;"> 21.36 </td>
   <td style="text-align:right;"> 34.50 </td>
   <td style="text-align:right;"> 955 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 272.23 </td>
   <td style="text-align:right;"> 196.04 </td>
   <td style="text-align:right;"> 23.73 </td>
   <td style="text-align:right;"> 37.38 </td>
   <td style="text-align:right;"> 955 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 284.61 </td>
   <td style="text-align:right;"> 485.23 </td>
   <td style="text-align:right;"> 26.21 </td>
   <td style="text-align:right;"> 39.91 </td>
   <td style="text-align:right;"> 955 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 259.89 </td>
   <td style="text-align:right;"> 583.91 </td>
   <td style="text-align:right;"> 28.24 </td>
   <td style="text-align:right;"> 42.38 </td>
   <td style="text-align:right;"> 953 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 753.86 </td>
   <td style="text-align:right;"> 2151.10 </td>
   <td style="text-align:right;"> 28.69 </td>
   <td style="text-align:right;"> 43.02 </td>
   <td style="text-align:right;"> 955 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 909.07 </td>
   <td style="text-align:right;"> 2309.24 </td>
   <td style="text-align:right;"> 31.45 </td>
   <td style="text-align:right;"> 46.42 </td>
   <td style="text-align:right;"> 955 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Alcúdia </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 816.68 </td>
   <td style="text-align:right;"> 2158.36 </td>
   <td style="text-align:right;"> 34.00 </td>
   <td style="text-align:right;"> 49.41 </td>
   <td style="text-align:right;"> 955 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 279.00 </td>
   <td style="text-align:right;"> 402.79 </td>
   <td style="text-align:right;"> 22.89 </td>
   <td style="text-align:right;"> 33.27 </td>
   <td style="text-align:right;"> 61 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 255.95 </td>
   <td style="text-align:right;"> 261.49 </td>
   <td style="text-align:right;"> 23.11 </td>
   <td style="text-align:right;"> 33.68 </td>
   <td style="text-align:right;"> 61 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 313.19 </td>
   <td style="text-align:right;"> 397.38 </td>
   <td style="text-align:right;"> 25.17 </td>
   <td style="text-align:right;"> 35.61 </td>
   <td style="text-align:right;"> 60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 310.71 </td>
   <td style="text-align:right;"> 403.79 </td>
   <td style="text-align:right;"> 27.45 </td>
   <td style="text-align:right;"> 36.98 </td>
   <td style="text-align:right;"> 60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 300.39 </td>
   <td style="text-align:right;"> 301.83 </td>
   <td style="text-align:right;"> 28.92 </td>
   <td style="text-align:right;"> 38.31 </td>
   <td style="text-align:right;"> 60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1372.37 </td>
   <td style="text-align:right;"> 3014.46 </td>
   <td style="text-align:right;"> 29.18 </td>
   <td style="text-align:right;"> 38.94 </td>
   <td style="text-align:right;"> 60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1536.37 </td>
   <td style="text-align:right;"> 3190.42 </td>
   <td style="text-align:right;"> 31.13 </td>
   <td style="text-align:right;"> 40.88 </td>
   <td style="text-align:right;"> 60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algaida </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1380.87 </td>
   <td style="text-align:right;"> 2952.53 </td>
   <td style="text-align:right;"> 33.47 </td>
   <td style="text-align:right;"> 42.71 </td>
   <td style="text-align:right;"> 60 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 586.34 </td>
   <td style="text-align:right;"> 1058.44 </td>
   <td style="text-align:right;"> 27.76 </td>
   <td style="text-align:right;"> 49.86 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 584.71 </td>
   <td style="text-align:right;"> 933.90 </td>
   <td style="text-align:right;"> 28.65 </td>
   <td style="text-align:right;"> 50.87 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 752.56 </td>
   <td style="text-align:right;"> 1253.32 </td>
   <td style="text-align:right;"> 31.03 </td>
   <td style="text-align:right;"> 52.96 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 795.60 </td>
   <td style="text-align:right;"> 1415.74 </td>
   <td style="text-align:right;"> 33.74 </td>
   <td style="text-align:right;"> 54.70 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 693.70 </td>
   <td style="text-align:right;"> 1350.73 </td>
   <td style="text-align:right;"> 35.54 </td>
   <td style="text-align:right;"> 56.09 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 930.99 </td>
   <td style="text-align:right;"> 1866.29 </td>
   <td style="text-align:right;"> 36.12 </td>
   <td style="text-align:right;"> 56.91 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1020.52 </td>
   <td style="text-align:right;"> 1844.39 </td>
   <td style="text-align:right;"> 38.69 </td>
   <td style="text-align:right;"> 59.57 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Andratx </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1120.12 </td>
   <td style="text-align:right;"> 2070.25 </td>
   <td style="text-align:right;"> 41.06 </td>
   <td style="text-align:right;"> 61.25 </td>
   <td style="text-align:right;"> 136 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 218.38 </td>
   <td style="text-align:right;"> 154.56 </td>
   <td style="text-align:right;"> 8.41 </td>
   <td style="text-align:right;"> 11.32 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 211.93 </td>
   <td style="text-align:right;"> 150.06 </td>
   <td style="text-align:right;"> 8.57 </td>
   <td style="text-align:right;"> 11.40 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 256.52 </td>
   <td style="text-align:right;"> 195.54 </td>
   <td style="text-align:right;"> 9.65 </td>
   <td style="text-align:right;"> 12.38 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 243.30 </td>
   <td style="text-align:right;"> 179.95 </td>
   <td style="text-align:right;"> 11.48 </td>
   <td style="text-align:right;"> 13.77 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 222.39 </td>
   <td style="text-align:right;"> 122.51 </td>
   <td style="text-align:right;"> 12.30 </td>
   <td style="text-align:right;"> 14.48 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 2705.76 </td>
   <td style="text-align:right;"> 4159.36 </td>
   <td style="text-align:right;"> 12.39 </td>
   <td style="text-align:right;"> 14.61 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 3069.37 </td>
   <td style="text-align:right;"> 4268.56 </td>
   <td style="text-align:right;"> 13.65 </td>
   <td style="text-align:right;"> 15.79 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ariany </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 2861.72 </td>
   <td style="text-align:right;"> 4136.70 </td>
   <td style="text-align:right;"> 15.20 </td>
   <td style="text-align:right;"> 17.46 </td>
   <td style="text-align:right;"> 46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 244.51 </td>
   <td style="text-align:right;"> 226.57 </td>
   <td style="text-align:right;"> 13.66 </td>
   <td style="text-align:right;"> 24.48 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 248.78 </td>
   <td style="text-align:right;"> 230.45 </td>
   <td style="text-align:right;"> 14.00 </td>
   <td style="text-align:right;"> 24.81 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 247.18 </td>
   <td style="text-align:right;"> 132.03 </td>
   <td style="text-align:right;"> 15.20 </td>
   <td style="text-align:right;"> 26.14 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 298.43 </td>
   <td style="text-align:right;"> 750.50 </td>
   <td style="text-align:right;"> 17.13 </td>
   <td style="text-align:right;"> 27.71 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 291.50 </td>
   <td style="text-align:right;"> 757.48 </td>
   <td style="text-align:right;"> 18.21 </td>
   <td style="text-align:right;"> 28.89 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1611.71 </td>
   <td style="text-align:right;"> 3306.08 </td>
   <td style="text-align:right;"> 18.50 </td>
   <td style="text-align:right;"> 29.29 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 2329.95 </td>
   <td style="text-align:right;"> 3843.43 </td>
   <td style="text-align:right;"> 19.88 </td>
   <td style="text-align:right;"> 30.69 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Artà </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1954.48 </td>
   <td style="text-align:right;"> 3570.04 </td>
   <td style="text-align:right;"> 21.66 </td>
   <td style="text-align:right;"> 32.14 </td>
   <td style="text-align:right;"> 173 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 213.86 </td>
   <td style="text-align:right;"> 161.27 </td>
   <td style="text-align:right;"> 64.41 </td>
   <td style="text-align:right;"> 84.45 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 219.20 </td>
   <td style="text-align:right;"> 168.24 </td>
   <td style="text-align:right;"> 66.25 </td>
   <td style="text-align:right;"> 86.80 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 258.55 </td>
   <td style="text-align:right;"> 233.55 </td>
   <td style="text-align:right;"> 70.50 </td>
   <td style="text-align:right;"> 90.22 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 257.61 </td>
   <td style="text-align:right;"> 169.13 </td>
   <td style="text-align:right;"> 75.52 </td>
   <td style="text-align:right;"> 93.93 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 244.64 </td>
   <td style="text-align:right;"> 206.85 </td>
   <td style="text-align:right;"> 78.75 </td>
   <td style="text-align:right;"> 96.97 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 666.24 </td>
   <td style="text-align:right;"> 2033.87 </td>
   <td style="text-align:right;"> 80.34 </td>
   <td style="text-align:right;"> 99.04 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 660.95 </td>
   <td style="text-align:right;"> 1849.59 </td>
   <td style="text-align:right;"> 85.57 </td>
   <td style="text-align:right;"> 102.83 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Banyalbufar </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 647.61 </td>
   <td style="text-align:right;"> 1849.15 </td>
   <td style="text-align:right;"> 91.18 </td>
   <td style="text-align:right;"> 107.34 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 259.07 </td>
   <td style="text-align:right;"> 194.74 </td>
   <td style="text-align:right;"> 26.07 </td>
   <td style="text-align:right;"> 39.77 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 264.00 </td>
   <td style="text-align:right;"> 193.16 </td>
   <td style="text-align:right;"> 26.48 </td>
   <td style="text-align:right;"> 40.20 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 281.20 </td>
   <td style="text-align:right;"> 198.79 </td>
   <td style="text-align:right;"> 28.16 </td>
   <td style="text-align:right;"> 41.95 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 294.39 </td>
   <td style="text-align:right;"> 221.81 </td>
   <td style="text-align:right;"> 30.82 </td>
   <td style="text-align:right;"> 44.04 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 281.25 </td>
   <td style="text-align:right;"> 256.97 </td>
   <td style="text-align:right;"> 32.39 </td>
   <td style="text-align:right;"> 45.37 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 969.56 </td>
   <td style="text-align:right;"> 2428.47 </td>
   <td style="text-align:right;"> 32.68 </td>
   <td style="text-align:right;"> 45.72 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1000.42 </td>
   <td style="text-align:right;"> 2372.78 </td>
   <td style="text-align:right;"> 34.96 </td>
   <td style="text-align:right;"> 47.59 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Binissalem </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1124.80 </td>
   <td style="text-align:right;"> 2600.39 </td>
   <td style="text-align:right;"> 37.52 </td>
   <td style="text-align:right;"> 50.31 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 288.28 </td>
   <td style="text-align:right;"> 230.47 </td>
   <td style="text-align:right;"> 35.47 </td>
   <td style="text-align:right;"> 48.23 </td>
   <td style="text-align:right;"> 43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 298.51 </td>
   <td style="text-align:right;"> 223.69 </td>
   <td style="text-align:right;"> 36.16 </td>
   <td style="text-align:right;"> 48.74 </td>
   <td style="text-align:right;"> 43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 354.19 </td>
   <td style="text-align:right;"> 269.21 </td>
   <td style="text-align:right;"> 38.81 </td>
   <td style="text-align:right;"> 50.50 </td>
   <td style="text-align:right;"> 43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 350.88 </td>
   <td style="text-align:right;"> 269.17 </td>
   <td style="text-align:right;"> 42.56 </td>
   <td style="text-align:right;"> 52.37 </td>
   <td style="text-align:right;"> 43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 316.35 </td>
   <td style="text-align:right;"> 271.64 </td>
   <td style="text-align:right;"> 44.77 </td>
   <td style="text-align:right;"> 54.05 </td>
   <td style="text-align:right;"> 43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 752.64 </td>
   <td style="text-align:right;"> 1832.92 </td>
   <td style="text-align:right;"> 46.29 </td>
   <td style="text-align:right;"> 55.04 </td>
   <td style="text-align:right;"> 42 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 786.49 </td>
   <td style="text-align:right;"> 1845.50 </td>
   <td style="text-align:right;"> 49.12 </td>
   <td style="text-align:right;"> 57.33 </td>
   <td style="text-align:right;"> 42 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bunyola </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 769.86 </td>
   <td style="text-align:right;"> 1823.79 </td>
   <td style="text-align:right;"> 53.33 </td>
   <td style="text-align:right;"> 59.50 </td>
   <td style="text-align:right;"> 42 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 254.64 </td>
   <td style="text-align:right;"> 146.96 </td>
   <td style="text-align:right;"> 10.94 </td>
   <td style="text-align:right;"> 21.53 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 252.61 </td>
   <td style="text-align:right;"> 145.21 </td>
   <td style="text-align:right;"> 11.07 </td>
   <td style="text-align:right;"> 21.74 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 297.86 </td>
   <td style="text-align:right;"> 171.24 </td>
   <td style="text-align:right;"> 11.92 </td>
   <td style="text-align:right;"> 22.80 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 305.36 </td>
   <td style="text-align:right;"> 182.89 </td>
   <td style="text-align:right;"> 13.20 </td>
   <td style="text-align:right;"> 24.64 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 283.96 </td>
   <td style="text-align:right;"> 168.52 </td>
   <td style="text-align:right;"> 13.91 </td>
   <td style="text-align:right;"> 25.49 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 851.95 </td>
   <td style="text-align:right;"> 2223.10 </td>
   <td style="text-align:right;"> 13.98 </td>
   <td style="text-align:right;"> 25.61 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 718.56 </td>
   <td style="text-align:right;"> 1838.50 </td>
   <td style="text-align:right;"> 14.84 </td>
   <td style="text-align:right;"> 26.56 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Búger </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 784.13 </td>
   <td style="text-align:right;"> 2025.52 </td>
   <td style="text-align:right;"> 16.09 </td>
   <td style="text-align:right;"> 28.38 </td>
   <td style="text-align:right;"> 98 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 462.89 </td>
   <td style="text-align:right;"> 600.58 </td>
   <td style="text-align:right;"> 38.69 </td>
   <td style="text-align:right;"> 91.97 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 448.31 </td>
   <td style="text-align:right;"> 543.37 </td>
   <td style="text-align:right;"> 40.19 </td>
   <td style="text-align:right;"> 96.68 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 508.48 </td>
   <td style="text-align:right;"> 490.07 </td>
   <td style="text-align:right;"> 45.68 </td>
   <td style="text-align:right;"> 110.76 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 510.79 </td>
   <td style="text-align:right;"> 533.10 </td>
   <td style="text-align:right;"> 51.07 </td>
   <td style="text-align:right;"> 122.70 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 451.72 </td>
   <td style="text-align:right;"> 510.44 </td>
   <td style="text-align:right;"> 54.77 </td>
   <td style="text-align:right;"> 129.53 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 740.86 </td>
   <td style="text-align:right;"> 1731.77 </td>
   <td style="text-align:right;"> 55.60 </td>
   <td style="text-align:right;"> 131.52 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 791.22 </td>
   <td style="text-align:right;"> 1538.36 </td>
   <td style="text-align:right;"> 60.68 </td>
   <td style="text-align:right;"> 139.63 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Calvià </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 833.80 </td>
   <td style="text-align:right;"> 1760.20 </td>
   <td style="text-align:right;"> 64.74 </td>
   <td style="text-align:right;"> 146.25 </td>
   <td style="text-align:right;"> 183 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 195.73 </td>
   <td style="text-align:right;"> 123.24 </td>
   <td style="text-align:right;"> 23.06 </td>
   <td style="text-align:right;"> 33.90 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 200.32 </td>
   <td style="text-align:right;"> 154.02 </td>
   <td style="text-align:right;"> 23.36 </td>
   <td style="text-align:right;"> 34.32 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 244.54 </td>
   <td style="text-align:right;"> 207.37 </td>
   <td style="text-align:right;"> 25.34 </td>
   <td style="text-align:right;"> 36.46 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 240.07 </td>
   <td style="text-align:right;"> 204.85 </td>
   <td style="text-align:right;"> 28.46 </td>
   <td style="text-align:right;"> 39.43 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 220.01 </td>
   <td style="text-align:right;"> 131.41 </td>
   <td style="text-align:right;"> 30.33 </td>
   <td style="text-align:right;"> 41.76 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1423.48 </td>
   <td style="text-align:right;"> 3129.20 </td>
   <td style="text-align:right;"> 30.46 </td>
   <td style="text-align:right;"> 41.82 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1365.45 </td>
   <td style="text-align:right;"> 2983.56 </td>
   <td style="text-align:right;"> 32.84 </td>
   <td style="text-align:right;"> 44.99 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campanet </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1299.56 </td>
   <td style="text-align:right;"> 2918.05 </td>
   <td style="text-align:right;"> 35.54 </td>
   <td style="text-align:right;"> 47.98 </td>
   <td style="text-align:right;"> 97 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 263.56 </td>
   <td style="text-align:right;"> 245.69 </td>
   <td style="text-align:right;"> 14.60 </td>
   <td style="text-align:right;"> 21.49 </td>
   <td style="text-align:right;"> 311 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 260.82 </td>
   <td style="text-align:right;"> 216.54 </td>
   <td style="text-align:right;"> 14.84 </td>
   <td style="text-align:right;"> 21.73 </td>
   <td style="text-align:right;"> 311 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 293.29 </td>
   <td style="text-align:right;"> 230.80 </td>
   <td style="text-align:right;"> 16.08 </td>
   <td style="text-align:right;"> 22.86 </td>
   <td style="text-align:right;"> 310 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 322.70 </td>
   <td style="text-align:right;"> 599.78 </td>
   <td style="text-align:right;"> 17.82 </td>
   <td style="text-align:right;"> 24.25 </td>
   <td style="text-align:right;"> 310 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 323.22 </td>
   <td style="text-align:right;"> 609.11 </td>
   <td style="text-align:right;"> 19.00 </td>
   <td style="text-align:right;"> 25.37 </td>
   <td style="text-align:right;"> 310 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1171.16 </td>
   <td style="text-align:right;"> 2777.68 </td>
   <td style="text-align:right;"> 19.12 </td>
   <td style="text-align:right;"> 25.60 </td>
   <td style="text-align:right;"> 311 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1455.01 </td>
   <td style="text-align:right;"> 3029.80 </td>
   <td style="text-align:right;"> 20.76 </td>
   <td style="text-align:right;"> 26.92 </td>
   <td style="text-align:right;"> 311 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Campos </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1346.72 </td>
   <td style="text-align:right;"> 2927.00 </td>
   <td style="text-align:right;"> 22.95 </td>
   <td style="text-align:right;"> 28.84 </td>
   <td style="text-align:right;"> 312 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 222.22 </td>
   <td style="text-align:right;"> 259.10 </td>
   <td style="text-align:right;"> 12.56 </td>
   <td style="text-align:right;"> 18.15 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 224.80 </td>
   <td style="text-align:right;"> 257.60 </td>
   <td style="text-align:right;"> 12.94 </td>
   <td style="text-align:right;"> 18.69 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 259.51 </td>
   <td style="text-align:right;"> 282.05 </td>
   <td style="text-align:right;"> 14.15 </td>
   <td style="text-align:right;"> 20.05 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 262.47 </td>
   <td style="text-align:right;"> 287.17 </td>
   <td style="text-align:right;"> 16.12 </td>
   <td style="text-align:right;"> 21.73 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 241.43 </td>
   <td style="text-align:right;"> 289.46 </td>
   <td style="text-align:right;"> 17.29 </td>
   <td style="text-align:right;"> 22.91 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 2503.65 </td>
   <td style="text-align:right;"> 3937.40 </td>
   <td style="text-align:right;"> 17.56 </td>
   <td style="text-align:right;"> 23.26 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 2879.30 </td>
   <td style="text-align:right;"> 4080.55 </td>
   <td style="text-align:right;"> 19.24 </td>
   <td style="text-align:right;"> 24.95 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Capdepera </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 2850.43 </td>
   <td style="text-align:right;"> 4079.32 </td>
   <td style="text-align:right;"> 21.37 </td>
   <td style="text-align:right;"> 26.76 </td>
   <td style="text-align:right;"> 281 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 253.14 </td>
   <td style="text-align:right;"> 135.75 </td>
   <td style="text-align:right;"> 42.43 </td>
   <td style="text-align:right;"> 41.27 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 253.71 </td>
   <td style="text-align:right;"> 125.93 </td>
   <td style="text-align:right;"> 42.71 </td>
   <td style="text-align:right;"> 41.68 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 288.71 </td>
   <td style="text-align:right;"> 131.98 </td>
   <td style="text-align:right;"> 46.00 </td>
   <td style="text-align:right;"> 44.31 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 260.86 </td>
   <td style="text-align:right;"> 136.74 </td>
   <td style="text-align:right;"> 51.00 </td>
   <td style="text-align:right;"> 46.12 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 252.29 </td>
   <td style="text-align:right;"> 169.25 </td>
   <td style="text-align:right;"> 53.86 </td>
   <td style="text-align:right;"> 47.86 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1622.86 </td>
   <td style="text-align:right;"> 3695.47 </td>
   <td style="text-align:right;"> 53.86 </td>
   <td style="text-align:right;"> 47.86 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 3376.50 </td>
   <td style="text-align:right;"> 4756.20 </td>
   <td style="text-align:right;"> 57.86 </td>
   <td style="text-align:right;"> 51.21 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Consell </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 2908.86 </td>
   <td style="text-align:right;"> 4513.86 </td>
   <td style="text-align:right;"> 62.43 </td>
   <td style="text-align:right;"> 54.66 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 185.13 </td>
   <td style="text-align:right;"> 118.01 </td>
   <td style="text-align:right;"> 16.23 </td>
   <td style="text-align:right;"> 22.69 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 197.29 </td>
   <td style="text-align:right;"> 122.89 </td>
   <td style="text-align:right;"> 16.58 </td>
   <td style="text-align:right;"> 22.96 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 231.68 </td>
   <td style="text-align:right;"> 152.09 </td>
   <td style="text-align:right;"> 18.26 </td>
   <td style="text-align:right;"> 24.45 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 241.48 </td>
   <td style="text-align:right;"> 149.57 </td>
   <td style="text-align:right;"> 20.68 </td>
   <td style="text-align:right;"> 25.83 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 204.35 </td>
   <td style="text-align:right;"> 127.13 </td>
   <td style="text-align:right;"> 22.58 </td>
   <td style="text-align:right;"> 27.20 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 2035.45 </td>
   <td style="text-align:right;"> 3723.50 </td>
   <td style="text-align:right;"> 23.35 </td>
   <td style="text-align:right;"> 27.49 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 2262.43 </td>
   <td style="text-align:right;"> 3793.18 </td>
   <td style="text-align:right;"> 25.52 </td>
   <td style="text-align:right;"> 28.95 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Costitx </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 2055.87 </td>
   <td style="text-align:right;"> 3715.32 </td>
   <td style="text-align:right;"> 27.74 </td>
   <td style="text-align:right;"> 30.47 </td>
   <td style="text-align:right;"> 31 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 462.50 </td>
   <td style="text-align:right;"> 368.94 </td>
   <td style="text-align:right;"> 40.35 </td>
   <td style="text-align:right;"> 46.99 </td>
   <td style="text-align:right;"> 43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 494.44 </td>
   <td style="text-align:right;"> 411.76 </td>
   <td style="text-align:right;"> 41.14 </td>
   <td style="text-align:right;"> 47.87 </td>
   <td style="text-align:right;"> 43 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 657.05 </td>
   <td style="text-align:right;"> 590.03 </td>
   <td style="text-align:right;"> 42.70 </td>
   <td style="text-align:right;"> 50.00 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 656.40 </td>
   <td style="text-align:right;"> 575.34 </td>
   <td style="text-align:right;"> 46.36 </td>
   <td style="text-align:right;"> 51.84 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 515.55 </td>
   <td style="text-align:right;"> 493.98 </td>
   <td style="text-align:right;"> 48.34 </td>
   <td style="text-align:right;"> 53.66 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 545.26 </td>
   <td style="text-align:right;"> 471.00 </td>
   <td style="text-align:right;"> 48.75 </td>
   <td style="text-align:right;"> 54.00 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 698.77 </td>
   <td style="text-align:right;"> 618.51 </td>
   <td style="text-align:right;"> 51.43 </td>
   <td style="text-align:right;"> 56.49 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Deyá </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 652.05 </td>
   <td style="text-align:right;"> 576.26 </td>
   <td style="text-align:right;"> 55.07 </td>
   <td style="text-align:right;"> 58.65 </td>
   <td style="text-align:right;"> 44 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 243.47 </td>
   <td style="text-align:right;"> 209.09 </td>
   <td style="text-align:right;"> 35.11 </td>
   <td style="text-align:right;"> 43.63 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 246.95 </td>
   <td style="text-align:right;"> 228.32 </td>
   <td style="text-align:right;"> 35.42 </td>
   <td style="text-align:right;"> 44.08 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 280.42 </td>
   <td style="text-align:right;"> 288.71 </td>
   <td style="text-align:right;"> 37.74 </td>
   <td style="text-align:right;"> 46.33 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 277.21 </td>
   <td style="text-align:right;"> 290.09 </td>
   <td style="text-align:right;"> 41.00 </td>
   <td style="text-align:right;"> 47.58 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 264.11 </td>
   <td style="text-align:right;"> 222.30 </td>
   <td style="text-align:right;"> 43.00 </td>
   <td style="text-align:right;"> 49.07 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 794.53 </td>
   <td style="text-align:right;"> 2128.49 </td>
   <td style="text-align:right;"> 43.21 </td>
   <td style="text-align:right;"> 49.29 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 240.75 </td>
   <td style="text-align:right;"> 145.48 </td>
   <td style="text-align:right;"> 45.32 </td>
   <td style="text-align:right;"> 50.85 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Escorca </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 296.16 </td>
   <td style="text-align:right;"> 288.55 </td>
   <td style="text-align:right;"> 47.84 </td>
   <td style="text-align:right;"> 52.78 </td>
   <td style="text-align:right;"> 19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 373.24 </td>
   <td style="text-align:right;"> 310.75 </td>
   <td style="text-align:right;"> 22.24 </td>
   <td style="text-align:right;"> 39.26 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 417.44 </td>
   <td style="text-align:right;"> 266.37 </td>
   <td style="text-align:right;"> 22.09 </td>
   <td style="text-align:right;"> 40.16 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 488.50 </td>
   <td style="text-align:right;"> 244.38 </td>
   <td style="text-align:right;"> 24.06 </td>
   <td style="text-align:right;"> 42.30 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 502.26 </td>
   <td style="text-align:right;"> 251.02 </td>
   <td style="text-align:right;"> 27.06 </td>
   <td style="text-align:right;"> 44.61 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 569.35 </td>
   <td style="text-align:right;"> 373.51 </td>
   <td style="text-align:right;"> 28.15 </td>
   <td style="text-align:right;"> 46.06 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 716.21 </td>
   <td style="text-align:right;"> 1686.61 </td>
   <td style="text-align:right;"> 28.38 </td>
   <td style="text-align:right;"> 46.80 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1018.65 </td>
   <td style="text-align:right;"> 2055.43 </td>
   <td style="text-align:right;"> 30.24 </td>
   <td style="text-align:right;"> 49.25 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Esporles </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 992.06 </td>
   <td style="text-align:right;"> 2043.64 </td>
   <td style="text-align:right;"> 32.29 </td>
   <td style="text-align:right;"> 51.28 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 514.33 </td>
   <td style="text-align:right;"> 390.84 </td>
   <td style="text-align:right;"> 19.50 </td>
   <td style="text-align:right;"> 40.75 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 242.39 </td>
   <td style="text-align:right;"> 165.94 </td>
   <td style="text-align:right;"> 19.89 </td>
   <td style="text-align:right;"> 41.50 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 297.72 </td>
   <td style="text-align:right;"> 186.27 </td>
   <td style="text-align:right;"> 21.33 </td>
   <td style="text-align:right;"> 43.73 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 1898.22 </td>
   <td style="text-align:right;"> 3731.47 </td>
   <td style="text-align:right;"> 23.61 </td>
   <td style="text-align:right;"> 45.35 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 2114.00 </td>
   <td style="text-align:right;"> 3647.73 </td>
   <td style="text-align:right;"> 25.00 </td>
   <td style="text-align:right;"> 47.13 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 766.50 </td>
   <td style="text-align:right;"> 2308.94 </td>
   <td style="text-align:right;"> 25.28 </td>
   <td style="text-align:right;"> 47.64 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1894.67 </td>
   <td style="text-align:right;"> 3742.63 </td>
   <td style="text-align:right;"> 27.11 </td>
   <td style="text-align:right;"> 51.28 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Estellencs </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1376.28 </td>
   <td style="text-align:right;"> 3142.08 </td>
   <td style="text-align:right;"> 29.00 </td>
   <td style="text-align:right;"> 53.54 </td>
   <td style="text-align:right;"> 18 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 351.82 </td>
   <td style="text-align:right;"> 378.65 </td>
   <td style="text-align:right;"> 17.83 </td>
   <td style="text-align:right;"> 31.98 </td>
   <td style="text-align:right;"> 387 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 337.74 </td>
   <td style="text-align:right;"> 365.72 </td>
   <td style="text-align:right;"> 18.08 </td>
   <td style="text-align:right;"> 32.52 </td>
   <td style="text-align:right;"> 387 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 327.79 </td>
   <td style="text-align:right;"> 266.58 </td>
   <td style="text-align:right;"> 19.90 </td>
   <td style="text-align:right;"> 34.12 </td>
   <td style="text-align:right;"> 387 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 342.51 </td>
   <td style="text-align:right;"> 550.95 </td>
   <td style="text-align:right;"> 22.37 </td>
   <td style="text-align:right;"> 35.72 </td>
   <td style="text-align:right;"> 387 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 394.52 </td>
   <td style="text-align:right;"> 621.52 </td>
   <td style="text-align:right;"> 23.93 </td>
   <td style="text-align:right;"> 37.26 </td>
   <td style="text-align:right;"> 387 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1502.98 </td>
   <td style="text-align:right;"> 2987.93 </td>
   <td style="text-align:right;"> 24.12 </td>
   <td style="text-align:right;"> 37.60 </td>
   <td style="text-align:right;"> 387 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1633.14 </td>
   <td style="text-align:right;"> 3155.24 </td>
   <td style="text-align:right;"> 26.46 </td>
   <td style="text-align:right;"> 39.78 </td>
   <td style="text-align:right;"> 388 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Felanitx </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1567.73 </td>
   <td style="text-align:right;"> 3121.42 </td>
   <td style="text-align:right;"> 28.98 </td>
   <td style="text-align:right;"> 41.60 </td>
   <td style="text-align:right;"> 388 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 245.16 </td>
   <td style="text-align:right;"> 233.47 </td>
   <td style="text-align:right;"> 44.77 </td>
   <td style="text-align:right;"> 51.80 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 249.38 </td>
   <td style="text-align:right;"> 220.01 </td>
   <td style="text-align:right;"> 45.21 </td>
   <td style="text-align:right;"> 52.73 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 248.15 </td>
   <td style="text-align:right;"> 139.33 </td>
   <td style="text-align:right;"> 48.03 </td>
   <td style="text-align:right;"> 54.13 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 245.15 </td>
   <td style="text-align:right;"> 127.88 </td>
   <td style="text-align:right;"> 51.76 </td>
   <td style="text-align:right;"> 56.34 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 221.58 </td>
   <td style="text-align:right;"> 127.55 </td>
   <td style="text-align:right;"> 54.48 </td>
   <td style="text-align:right;"> 58.44 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1040.91 </td>
   <td style="text-align:right;"> 2659.07 </td>
   <td style="text-align:right;"> 55.12 </td>
   <td style="text-align:right;"> 59.02 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1125.00 </td>
   <td style="text-align:right;"> 2714.83 </td>
   <td style="text-align:right;"> 58.69 </td>
   <td style="text-align:right;"> 61.59 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Fornalutx </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1357.33 </td>
   <td style="text-align:right;"> 2970.11 </td>
   <td style="text-align:right;"> 62.74 </td>
   <td style="text-align:right;"> 63.39 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 241.46 </td>
   <td style="text-align:right;"> 165.96 </td>
   <td style="text-align:right;"> 16.27 </td>
   <td style="text-align:right;"> 26.82 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 246.27 </td>
   <td style="text-align:right;"> 176.19 </td>
   <td style="text-align:right;"> 16.61 </td>
   <td style="text-align:right;"> 27.64 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 278.39 </td>
   <td style="text-align:right;"> 243.25 </td>
   <td style="text-align:right;"> 17.90 </td>
   <td style="text-align:right;"> 28.77 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 272.87 </td>
   <td style="text-align:right;"> 243.34 </td>
   <td style="text-align:right;"> 19.95 </td>
   <td style="text-align:right;"> 30.93 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 249.85 </td>
   <td style="text-align:right;"> 139.94 </td>
   <td style="text-align:right;"> 20.93 </td>
   <td style="text-align:right;"> 31.88 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 769.53 </td>
   <td style="text-align:right;"> 2144.98 </td>
   <td style="text-align:right;"> 21.29 </td>
   <td style="text-align:right;"> 32.58 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 790.43 </td>
   <td style="text-align:right;"> 2116.63 </td>
   <td style="text-align:right;"> 22.71 </td>
   <td style="text-align:right;"> 34.20 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Inca </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 866.87 </td>
   <td style="text-align:right;"> 2230.29 </td>
   <td style="text-align:right;"> 24.73 </td>
   <td style="text-align:right;"> 35.97 </td>
   <td style="text-align:right;"> 143 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 191.46 </td>
   <td style="text-align:right;"> 80.50 </td>
   <td style="text-align:right;"> 23.92 </td>
   <td style="text-align:right;"> 52.42 </td>
   <td style="text-align:right;"> 26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 187.27 </td>
   <td style="text-align:right;"> 65.89 </td>
   <td style="text-align:right;"> 24.38 </td>
   <td style="text-align:right;"> 53.60 </td>
   <td style="text-align:right;"> 26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 223.96 </td>
   <td style="text-align:right;"> 86.18 </td>
   <td style="text-align:right;"> 24.89 </td>
   <td style="text-align:right;"> 54.78 </td>
   <td style="text-align:right;"> 27 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 215.81 </td>
   <td style="text-align:right;"> 71.12 </td>
   <td style="text-align:right;"> 26.52 </td>
   <td style="text-align:right;"> 56.71 </td>
   <td style="text-align:right;"> 27 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 200.41 </td>
   <td style="text-align:right;"> 94.75 </td>
   <td style="text-align:right;"> 27.44 </td>
   <td style="text-align:right;"> 57.99 </td>
   <td style="text-align:right;"> 27 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 200.04 </td>
   <td style="text-align:right;"> 88.05 </td>
   <td style="text-align:right;"> 27.81 </td>
   <td style="text-align:right;"> 59.24 </td>
   <td style="text-align:right;"> 27 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 239.52 </td>
   <td style="text-align:right;"> 95.96 </td>
   <td style="text-align:right;"> 29.44 </td>
   <td style="text-align:right;"> 60.75 </td>
   <td style="text-align:right;"> 27 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloret de Vistalegre </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 232.74 </td>
   <td style="text-align:right;"> 93.54 </td>
   <td style="text-align:right;"> 31.11 </td>
   <td style="text-align:right;"> 62.21 </td>
   <td style="text-align:right;"> 27 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 246.18 </td>
   <td style="text-align:right;"> 180.33 </td>
   <td style="text-align:right;"> 21.61 </td>
   <td style="text-align:right;"> 27.86 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 241.88 </td>
   <td style="text-align:right;"> 155.03 </td>
   <td style="text-align:right;"> 22.00 </td>
   <td style="text-align:right;"> 28.20 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 282.24 </td>
   <td style="text-align:right;"> 144.42 </td>
   <td style="text-align:right;"> 24.15 </td>
   <td style="text-align:right;"> 30.68 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 284.42 </td>
   <td style="text-align:right;"> 130.40 </td>
   <td style="text-align:right;"> 26.64 </td>
   <td style="text-align:right;"> 32.73 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 293.09 </td>
   <td style="text-align:right;"> 198.65 </td>
   <td style="text-align:right;"> 28.36 </td>
   <td style="text-align:right;"> 35.02 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 824.73 </td>
   <td style="text-align:right;"> 2246.55 </td>
   <td style="text-align:right;"> 29.18 </td>
   <td style="text-align:right;"> 35.92 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1132.36 </td>
   <td style="text-align:right;"> 2641.08 </td>
   <td style="text-align:right;"> 32.15 </td>
   <td style="text-align:right;"> 38.35 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Lloseta </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1089.85 </td>
   <td style="text-align:right;"> 2543.63 </td>
   <td style="text-align:right;"> 34.76 </td>
   <td style="text-align:right;"> 40.41 </td>
   <td style="text-align:right;"> 33 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 208.07 </td>
   <td style="text-align:right;"> 239.19 </td>
   <td style="text-align:right;"> 19.68 </td>
   <td style="text-align:right;"> 29.42 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 184.98 </td>
   <td style="text-align:right;"> 153.39 </td>
   <td style="text-align:right;"> 19.96 </td>
   <td style="text-align:right;"> 29.98 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 217.59 </td>
   <td style="text-align:right;"> 155.74 </td>
   <td style="text-align:right;"> 21.57 </td>
   <td style="text-align:right;"> 30.80 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 226.07 </td>
   <td style="text-align:right;"> 190.76 </td>
   <td style="text-align:right;"> 23.91 </td>
   <td style="text-align:right;"> 32.07 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 233.79 </td>
   <td style="text-align:right;"> 264.84 </td>
   <td style="text-align:right;"> 25.45 </td>
   <td style="text-align:right;"> 33.25 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 552.25 </td>
   <td style="text-align:right;"> 1720.70 </td>
   <td style="text-align:right;"> 25.66 </td>
   <td style="text-align:right;"> 33.48 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 598.53 </td>
   <td style="text-align:right;"> 1693.28 </td>
   <td style="text-align:right;"> 27.88 </td>
   <td style="text-align:right;"> 35.21 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llubí </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 448.89 </td>
   <td style="text-align:right;"> 1345.08 </td>
   <td style="text-align:right;"> 30.36 </td>
   <td style="text-align:right;"> 36.53 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 321.78 </td>
   <td style="text-align:right;"> 487.35 </td>
   <td style="text-align:right;"> 22.46 </td>
   <td style="text-align:right;"> 33.04 </td>
   <td style="text-align:right;"> 314 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 258.20 </td>
   <td style="text-align:right;"> 242.70 </td>
   <td style="text-align:right;"> 22.96 </td>
   <td style="text-align:right;"> 33.97 </td>
   <td style="text-align:right;"> 314 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 313.47 </td>
   <td style="text-align:right;"> 239.32 </td>
   <td style="text-align:right;"> 25.19 </td>
   <td style="text-align:right;"> 36.02 </td>
   <td style="text-align:right;"> 315 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 367.16 </td>
   <td style="text-align:right;"> 808.69 </td>
   <td style="text-align:right;"> 28.10 </td>
   <td style="text-align:right;"> 38.20 </td>
   <td style="text-align:right;"> 315 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 372.19 </td>
   <td style="text-align:right;"> 988.66 </td>
   <td style="text-align:right;"> 29.70 </td>
   <td style="text-align:right;"> 40.17 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1178.62 </td>
   <td style="text-align:right;"> 2721.52 </td>
   <td style="text-align:right;"> 30.04 </td>
   <td style="text-align:right;"> 40.87 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1622.16 </td>
   <td style="text-align:right;"> 3140.47 </td>
   <td style="text-align:right;"> 32.48 </td>
   <td style="text-align:right;"> 43.42 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Llucmajor </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1624.29 </td>
   <td style="text-align:right;"> 3161.08 </td>
   <td style="text-align:right;"> 35.26 </td>
   <td style="text-align:right;"> 45.81 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 251.01 </td>
   <td style="text-align:right;"> 212.64 </td>
   <td style="text-align:right;"> 16.19 </td>
   <td style="text-align:right;"> 25.80 </td>
   <td style="text-align:right;"> 442 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 247.37 </td>
   <td style="text-align:right;"> 200.90 </td>
   <td style="text-align:right;"> 16.60 </td>
   <td style="text-align:right;"> 26.40 </td>
   <td style="text-align:right;"> 442 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 285.84 </td>
   <td style="text-align:right;"> 221.22 </td>
   <td style="text-align:right;"> 18.36 </td>
   <td style="text-align:right;"> 28.36 </td>
   <td style="text-align:right;"> 442 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 350.51 </td>
   <td style="text-align:right;"> 833.99 </td>
   <td style="text-align:right;"> 20.87 </td>
   <td style="text-align:right;"> 30.70 </td>
   <td style="text-align:right;"> 443 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 340.49 </td>
   <td style="text-align:right;"> 844.28 </td>
   <td style="text-align:right;"> 22.49 </td>
   <td style="text-align:right;"> 32.74 </td>
   <td style="text-align:right;"> 443 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1101.70 </td>
   <td style="text-align:right;"> 2665.70 </td>
   <td style="text-align:right;"> 22.82 </td>
   <td style="text-align:right;"> 33.35 </td>
   <td style="text-align:right;"> 443 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1170.63 </td>
   <td style="text-align:right;"> 2704.70 </td>
   <td style="text-align:right;"> 24.95 </td>
   <td style="text-align:right;"> 36.42 </td>
   <td style="text-align:right;"> 443 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Manacor </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1178.35 </td>
   <td style="text-align:right;"> 2725.72 </td>
   <td style="text-align:right;"> 27.43 </td>
   <td style="text-align:right;"> 38.87 </td>
   <td style="text-align:right;"> 443 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 219.89 </td>
   <td style="text-align:right;"> 100.09 </td>
   <td style="text-align:right;"> 27.89 </td>
   <td style="text-align:right;"> 37.27 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 216.50 </td>
   <td style="text-align:right;"> 85.38 </td>
   <td style="text-align:right;"> 28.39 </td>
   <td style="text-align:right;"> 38.07 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 275.36 </td>
   <td style="text-align:right;"> 106.58 </td>
   <td style="text-align:right;"> 29.75 </td>
   <td style="text-align:right;"> 39.61 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 276.14 </td>
   <td style="text-align:right;"> 107.48 </td>
   <td style="text-align:right;"> 31.86 </td>
   <td style="text-align:right;"> 41.43 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 264.33 </td>
   <td style="text-align:right;"> 117.68 </td>
   <td style="text-align:right;"> 33.64 </td>
   <td style="text-align:right;"> 43.29 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 388.14 </td>
   <td style="text-align:right;"> 628.63 </td>
   <td style="text-align:right;"> 33.93 </td>
   <td style="text-align:right;"> 43.81 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 301.54 </td>
   <td style="text-align:right;"> 108.66 </td>
   <td style="text-align:right;"> 35.64 </td>
   <td style="text-align:right;"> 45.62 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mancor de la Vall </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 289.79 </td>
   <td style="text-align:right;"> 117.18 </td>
   <td style="text-align:right;"> 37.57 </td>
   <td style="text-align:right;"> 47.08 </td>
   <td style="text-align:right;"> 28 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 160.85 </td>
   <td style="text-align:right;"> 73.36 </td>
   <td style="text-align:right;"> 18.40 </td>
   <td style="text-align:right;"> 38.20 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 177.32 </td>
   <td style="text-align:right;"> 85.70 </td>
   <td style="text-align:right;"> 19.10 </td>
   <td style="text-align:right;"> 39.88 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 212.47 </td>
   <td style="text-align:right;"> 94.60 </td>
   <td style="text-align:right;"> 20.60 </td>
   <td style="text-align:right;"> 41.12 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 224.50 </td>
   <td style="text-align:right;"> 116.77 </td>
   <td style="text-align:right;"> 22.55 </td>
   <td style="text-align:right;"> 42.87 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 209.30 </td>
   <td style="text-align:right;"> 109.29 </td>
   <td style="text-align:right;"> 23.90 </td>
   <td style="text-align:right;"> 44.40 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 903.59 </td>
   <td style="text-align:right;"> 2567.45 </td>
   <td style="text-align:right;"> 24.45 </td>
   <td style="text-align:right;"> 45.56 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 901.46 </td>
   <td style="text-align:right;"> 2369.66 </td>
   <td style="text-align:right;"> 26.27 </td>
   <td style="text-align:right;"> 47.45 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Maria de la Salut </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 893.32 </td>
   <td style="text-align:right;"> 2339.80 </td>
   <td style="text-align:right;"> 27.90 </td>
   <td style="text-align:right;"> 48.15 </td>
   <td style="text-align:right;"> 40 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 351.10 </td>
   <td style="text-align:right;"> 243.92 </td>
   <td style="text-align:right;"> 24.52 </td>
   <td style="text-align:right;"> 29.05 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 298.16 </td>
   <td style="text-align:right;"> 131.77 </td>
   <td style="text-align:right;"> 24.81 </td>
   <td style="text-align:right;"> 29.31 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 365.49 </td>
   <td style="text-align:right;"> 173.20 </td>
   <td style="text-align:right;"> 26.97 </td>
   <td style="text-align:right;"> 31.06 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 520.44 </td>
   <td style="text-align:right;"> 1236.50 </td>
   <td style="text-align:right;"> 30.37 </td>
   <td style="text-align:right;"> 33.30 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 490.47 </td>
   <td style="text-align:right;"> 1262.20 </td>
   <td style="text-align:right;"> 31.95 </td>
   <td style="text-align:right;"> 34.65 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 2024.03 </td>
   <td style="text-align:right;"> 3569.59 </td>
   <td style="text-align:right;"> 32.24 </td>
   <td style="text-align:right;"> 34.88 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1566.71 </td>
   <td style="text-align:right;"> 3080.42 </td>
   <td style="text-align:right;"> 34.87 </td>
   <td style="text-align:right;"> 36.62 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Marratxí </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1761.22 </td>
   <td style="text-align:right;"> 3182.08 </td>
   <td style="text-align:right;"> 38.16 </td>
   <td style="text-align:right;"> 38.15 </td>
   <td style="text-align:right;"> 63 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 165.34 </td>
   <td style="text-align:right;"> 99.24 </td>
   <td style="text-align:right;"> 26.33 </td>
   <td style="text-align:right;"> 38.13 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 175.00 </td>
   <td style="text-align:right;"> 97.31 </td>
   <td style="text-align:right;"> 26.90 </td>
   <td style="text-align:right;"> 39.10 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 224.73 </td>
   <td style="text-align:right;"> 115.82 </td>
   <td style="text-align:right;"> 27.93 </td>
   <td style="text-align:right;"> 40.22 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 235.41 </td>
   <td style="text-align:right;"> 120.16 </td>
   <td style="text-align:right;"> 30.40 </td>
   <td style="text-align:right;"> 41.50 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 221.17 </td>
   <td style="text-align:right;"> 182.26 </td>
   <td style="text-align:right;"> 31.67 </td>
   <td style="text-align:right;"> 42.79 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 209.93 </td>
   <td style="text-align:right;"> 155.40 </td>
   <td style="text-align:right;"> 31.97 </td>
   <td style="text-align:right;"> 43.00 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 561.90 </td>
   <td style="text-align:right;"> 1787.66 </td>
   <td style="text-align:right;"> 33.73 </td>
   <td style="text-align:right;"> 45.50 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Montuïri </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 232.97 </td>
   <td style="text-align:right;"> 136.24 </td>
   <td style="text-align:right;"> 36.17 </td>
   <td style="text-align:right;"> 47.21 </td>
   <td style="text-align:right;"> 30 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 222.21 </td>
   <td style="text-align:right;"> 125.40 </td>
   <td style="text-align:right;"> 12.63 </td>
   <td style="text-align:right;"> 20.04 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 227.06 </td>
   <td style="text-align:right;"> 128.03 </td>
   <td style="text-align:right;"> 12.83 </td>
   <td style="text-align:right;"> 20.20 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 270.53 </td>
   <td style="text-align:right;"> 150.93 </td>
   <td style="text-align:right;"> 14.37 </td>
   <td style="text-align:right;"> 21.09 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 353.45 </td>
   <td style="text-align:right;"> 941.00 </td>
   <td style="text-align:right;"> 16.09 </td>
   <td style="text-align:right;"> 22.15 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 329.51 </td>
   <td style="text-align:right;"> 945.99 </td>
   <td style="text-align:right;"> 17.21 </td>
   <td style="text-align:right;"> 23.02 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 582.92 </td>
   <td style="text-align:right;"> 1792.93 </td>
   <td style="text-align:right;"> 17.36 </td>
   <td style="text-align:right;"> 23.20 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 814.51 </td>
   <td style="text-align:right;"> 2135.79 </td>
   <td style="text-align:right;"> 19.11 </td>
   <td style="text-align:right;"> 24.33 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Muro </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 849.41 </td>
   <td style="text-align:right;"> 2195.48 </td>
   <td style="text-align:right;"> 20.99 </td>
   <td style="text-align:right;"> 25.81 </td>
   <td style="text-align:right;"> 219 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 268.57 </td>
   <td style="text-align:right;"> 382.26 </td>
   <td style="text-align:right;"> 57.93 </td>
   <td style="text-align:right;"> 81.87 </td>
   <td style="text-align:right;"> 503 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 226.41 </td>
   <td style="text-align:right;"> 199.31 </td>
   <td style="text-align:right;"> 60.26 </td>
   <td style="text-align:right;"> 84.43 </td>
   <td style="text-align:right;"> 503 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 296.85 </td>
   <td style="text-align:right;"> 249.05 </td>
   <td style="text-align:right;"> 64.94 </td>
   <td style="text-align:right;"> 88.14 </td>
   <td style="text-align:right;"> 503 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 312.27 </td>
   <td style="text-align:right;"> 529.55 </td>
   <td style="text-align:right;"> 70.08 </td>
   <td style="text-align:right;"> 91.49 </td>
   <td style="text-align:right;"> 503 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 274.16 </td>
   <td style="text-align:right;"> 550.10 </td>
   <td style="text-align:right;"> 74.09 </td>
   <td style="text-align:right;"> 94.62 </td>
   <td style="text-align:right;"> 505 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 442.76 </td>
   <td style="text-align:right;"> 1339.07 </td>
   <td style="text-align:right;"> 76.13 </td>
   <td style="text-align:right;"> 96.70 </td>
   <td style="text-align:right;"> 505 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 519.51 </td>
   <td style="text-align:right;"> 1351.50 </td>
   <td style="text-align:right;"> 81.54 </td>
   <td style="text-align:right;"> 101.34 </td>
   <td style="text-align:right;"> 505 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Palma de Mallorca </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 517.49 </td>
   <td style="text-align:right;"> 1377.66 </td>
   <td style="text-align:right;"> 86.36 </td>
   <td style="text-align:right;"> 104.48 </td>
   <td style="text-align:right;"> 505 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 275.97 </td>
   <td style="text-align:right;"> 190.70 </td>
   <td style="text-align:right;"> 15.64 </td>
   <td style="text-align:right;"> 31.73 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 269.87 </td>
   <td style="text-align:right;"> 173.96 </td>
   <td style="text-align:right;"> 15.91 </td>
   <td style="text-align:right;"> 32.31 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 293.43 </td>
   <td style="text-align:right;"> 161.77 </td>
   <td style="text-align:right;"> 16.99 </td>
   <td style="text-align:right;"> 33.40 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 312.08 </td>
   <td style="text-align:right;"> 204.31 </td>
   <td style="text-align:right;"> 19.26 </td>
   <td style="text-align:right;"> 34.80 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 292.37 </td>
   <td style="text-align:right;"> 222.17 </td>
   <td style="text-align:right;"> 20.54 </td>
   <td style="text-align:right;"> 36.55 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1858.84 </td>
   <td style="text-align:right;"> 3496.28 </td>
   <td style="text-align:right;"> 20.83 </td>
   <td style="text-align:right;"> 37.20 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 2053.60 </td>
   <td style="text-align:right;"> 3629.39 </td>
   <td style="text-align:right;"> 22.21 </td>
   <td style="text-align:right;"> 39.18 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Petra </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1584.40 </td>
   <td style="text-align:right;"> 3182.15 </td>
   <td style="text-align:right;"> 24.26 </td>
   <td style="text-align:right;"> 40.85 </td>
   <td style="text-align:right;"> 78 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 270.42 </td>
   <td style="text-align:right;"> 487.09 </td>
   <td style="text-align:right;"> 13.75 </td>
   <td style="text-align:right;"> 24.97 </td>
   <td style="text-align:right;"> 1593 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 249.96 </td>
   <td style="text-align:right;"> 252.47 </td>
   <td style="text-align:right;"> 14.01 </td>
   <td style="text-align:right;"> 25.54 </td>
   <td style="text-align:right;"> 1593 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 319.14 </td>
   <td style="text-align:right;"> 353.99 </td>
   <td style="text-align:right;"> 15.62 </td>
   <td style="text-align:right;"> 26.90 </td>
   <td style="text-align:right;"> 1592 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 303.21 </td>
   <td style="text-align:right;"> 363.51 </td>
   <td style="text-align:right;"> 17.29 </td>
   <td style="text-align:right;"> 28.27 </td>
   <td style="text-align:right;"> 1593 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 288.70 </td>
   <td style="text-align:right;"> 394.20 </td>
   <td style="text-align:right;"> 18.51 </td>
   <td style="text-align:right;"> 29.39 </td>
   <td style="text-align:right;"> 1595 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 965.46 </td>
   <td style="text-align:right;"> 2455.13 </td>
   <td style="text-align:right;"> 18.73 </td>
   <td style="text-align:right;"> 29.88 </td>
   <td style="text-align:right;"> 1593 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1262.98 </td>
   <td style="text-align:right;"> 2777.98 </td>
   <td style="text-align:right;"> 20.47 </td>
   <td style="text-align:right;"> 31.45 </td>
   <td style="text-align:right;"> 1593 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Pollença </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1242.19 </td>
   <td style="text-align:right;"> 2741.79 </td>
   <td style="text-align:right;"> 22.23 </td>
   <td style="text-align:right;"> 32.95 </td>
   <td style="text-align:right;"> 1593 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 298.34 </td>
   <td style="text-align:right;"> 481.94 </td>
   <td style="text-align:right;"> 22.83 </td>
   <td style="text-align:right;"> 44.23 </td>
   <td style="text-align:right;"> 53 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 240.62 </td>
   <td style="text-align:right;"> 260.35 </td>
   <td style="text-align:right;"> 23.45 </td>
   <td style="text-align:right;"> 45.54 </td>
   <td style="text-align:right;"> 53 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 254.13 </td>
   <td style="text-align:right;"> 168.15 </td>
   <td style="text-align:right;"> 24.85 </td>
   <td style="text-align:right;"> 46.78 </td>
   <td style="text-align:right;"> 54 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 432.96 </td>
   <td style="text-align:right;"> 1335.36 </td>
   <td style="text-align:right;"> 27.24 </td>
   <td style="text-align:right;"> 47.81 </td>
   <td style="text-align:right;"> 54 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 486.09 </td>
   <td style="text-align:right;"> 1397.63 </td>
   <td style="text-align:right;"> 28.54 </td>
   <td style="text-align:right;"> 49.38 </td>
   <td style="text-align:right;"> 54 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1212.04 </td>
   <td style="text-align:right;"> 2934.00 </td>
   <td style="text-align:right;"> 29.49 </td>
   <td style="text-align:right;"> 50.47 </td>
   <td style="text-align:right;"> 53 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1006.43 </td>
   <td style="text-align:right;"> 2601.50 </td>
   <td style="text-align:right;"> 31.17 </td>
   <td style="text-align:right;"> 52.17 </td>
   <td style="text-align:right;"> 53 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Porreres </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1154.74 </td>
   <td style="text-align:right;"> 2822.62 </td>
   <td style="text-align:right;"> 33.68 </td>
   <td style="text-align:right;"> 53.68 </td>
   <td style="text-align:right;"> 53 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 247.87 </td>
   <td style="text-align:right;"> 147.55 </td>
   <td style="text-align:right;"> 62.39 </td>
   <td style="text-align:right;"> 79.72 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 274.04 </td>
   <td style="text-align:right;"> 159.92 </td>
   <td style="text-align:right;"> 63.09 </td>
   <td style="text-align:right;"> 80.30 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 279.62 </td>
   <td style="text-align:right;"> 160.06 </td>
   <td style="text-align:right;"> 66.48 </td>
   <td style="text-align:right;"> 83.74 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 713.68 </td>
   <td style="text-align:right;"> 2078.57 </td>
   <td style="text-align:right;"> 70.65 </td>
   <td style="text-align:right;"> 87.19 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 687.00 </td>
   <td style="text-align:right;"> 2038.91 </td>
   <td style="text-align:right;"> 73.30 </td>
   <td style="text-align:right;"> 88.94 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1051.74 </td>
   <td style="text-align:right;"> 2672.62 </td>
   <td style="text-align:right;"> 73.74 </td>
   <td style="text-align:right;"> 89.21 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1161.22 </td>
   <td style="text-align:right;"> 2792.91 </td>
   <td style="text-align:right;"> 76.74 </td>
   <td style="text-align:right;"> 91.55 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Puigpunyent </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1130.13 </td>
   <td style="text-align:right;"> 2802.04 </td>
   <td style="text-align:right;"> 80.22 </td>
   <td style="text-align:right;"> 93.17 </td>
   <td style="text-align:right;"> 23 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 296.75 </td>
   <td style="text-align:right;"> 765.26 </td>
   <td style="text-align:right;"> 17.09 </td>
   <td style="text-align:right;"> 30.62 </td>
   <td style="text-align:right;"> 171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 220.55 </td>
   <td style="text-align:right;"> 120.43 </td>
   <td style="text-align:right;"> 17.37 </td>
   <td style="text-align:right;"> 31.26 </td>
   <td style="text-align:right;"> 171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 274.99 </td>
   <td style="text-align:right;"> 160.03 </td>
   <td style="text-align:right;"> 18.97 </td>
   <td style="text-align:right;"> 32.82 </td>
   <td style="text-align:right;"> 171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 380.95 </td>
   <td style="text-align:right;"> 1063.66 </td>
   <td style="text-align:right;"> 20.76 </td>
   <td style="text-align:right;"> 34.08 </td>
   <td style="text-align:right;"> 171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 376.96 </td>
   <td style="text-align:right;"> 1075.20 </td>
   <td style="text-align:right;"> 21.81 </td>
   <td style="text-align:right;"> 35.44 </td>
   <td style="text-align:right;"> 171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 852.52 </td>
   <td style="text-align:right;"> 2295.81 </td>
   <td style="text-align:right;"> 22.01 </td>
   <td style="text-align:right;"> 35.81 </td>
   <td style="text-align:right;"> 171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 922.50 </td>
   <td style="text-align:right;"> 2318.16 </td>
   <td style="text-align:right;"> 23.47 </td>
   <td style="text-align:right;"> 37.13 </td>
   <td style="text-align:right;"> 171 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sa Pobla </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 796.82 </td>
   <td style="text-align:right;"> 2088.67 </td>
   <td style="text-align:right;"> 24.84 </td>
   <td style="text-align:right;"> 38.12 </td>
   <td style="text-align:right;"> 170 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 202.75 </td>
   <td style="text-align:right;"> 102.16 </td>
   <td style="text-align:right;"> 14.29 </td>
   <td style="text-align:right;"> 22.56 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 189.92 </td>
   <td style="text-align:right;"> 81.91 </td>
   <td style="text-align:right;"> 14.58 </td>
   <td style="text-align:right;"> 22.64 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 206.17 </td>
   <td style="text-align:right;"> 97.25 </td>
   <td style="text-align:right;"> 16.62 </td>
   <td style="text-align:right;"> 23.56 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 210.52 </td>
   <td style="text-align:right;"> 95.52 </td>
   <td style="text-align:right;"> 19.12 </td>
   <td style="text-align:right;"> 24.74 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 257.82 </td>
   <td style="text-align:right;"> 144.53 </td>
   <td style="text-align:right;"> 20.54 </td>
   <td style="text-align:right;"> 25.62 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 642.70 </td>
   <td style="text-align:right;"> 2042.45 </td>
   <td style="text-align:right;"> 20.88 </td>
   <td style="text-align:right;"> 25.68 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 581.17 </td>
   <td style="text-align:right;"> 1619.45 </td>
   <td style="text-align:right;"> 23.08 </td>
   <td style="text-align:right;"> 26.42 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Joan </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 559.08 </td>
   <td style="text-align:right;"> 1587.09 </td>
   <td style="text-align:right;"> 25.50 </td>
   <td style="text-align:right;"> 27.97 </td>
   <td style="text-align:right;"> 24 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 210.97 </td>
   <td style="text-align:right;"> 175.20 </td>
   <td style="text-align:right;"> 21.91 </td>
   <td style="text-align:right;"> 33.05 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 209.29 </td>
   <td style="text-align:right;"> 173.63 </td>
   <td style="text-align:right;"> 22.39 </td>
   <td style="text-align:right;"> 33.75 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 259.39 </td>
   <td style="text-align:right;"> 217.11 </td>
   <td style="text-align:right;"> 24.88 </td>
   <td style="text-align:right;"> 37.61 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 322.72 </td>
   <td style="text-align:right;"> 816.41 </td>
   <td style="text-align:right;"> 28.02 </td>
   <td style="text-align:right;"> 42.82 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 292.77 </td>
   <td style="text-align:right;"> 809.08 </td>
   <td style="text-align:right;"> 30.05 </td>
   <td style="text-align:right;"> 47.20 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 892.55 </td>
   <td style="text-align:right;"> 2420.32 </td>
   <td style="text-align:right;"> 30.45 </td>
   <td style="text-align:right;"> 47.66 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1104.73 </td>
   <td style="text-align:right;"> 2640.87 </td>
   <td style="text-align:right;"> 32.98 </td>
   <td style="text-align:right;"> 52.18 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sant Llorenç des Cardassar </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1077.53 </td>
   <td style="text-align:right;"> 2620.49 </td>
   <td style="text-align:right;"> 35.80 </td>
   <td style="text-align:right;"> 56.16 </td>
   <td style="text-align:right;"> 157 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 258.17 </td>
   <td style="text-align:right;"> 145.59 </td>
   <td style="text-align:right;"> 28.50 </td>
   <td style="text-align:right;"> 38.42 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 266.00 </td>
   <td style="text-align:right;"> 147.18 </td>
   <td style="text-align:right;"> 28.83 </td>
   <td style="text-align:right;"> 39.16 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 282.17 </td>
   <td style="text-align:right;"> 129.55 </td>
   <td style="text-align:right;"> 29.67 </td>
   <td style="text-align:right;"> 40.06 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 282.33 </td>
   <td style="text-align:right;"> 132.78 </td>
   <td style="text-align:right;"> 32.50 </td>
   <td style="text-align:right;"> 43.22 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 227.00 </td>
   <td style="text-align:right;"> 140.88 </td>
   <td style="text-align:right;"> 33.33 </td>
   <td style="text-align:right;"> 44.46 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1693.33 </td>
   <td style="text-align:right;"> 3580.63 </td>
   <td style="text-align:right;"> 33.50 </td>
   <td style="text-align:right;"> 44.83 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 2054.20 </td>
   <td style="text-align:right;"> 3883.77 </td>
   <td style="text-align:right;"> 34.33 </td>
   <td style="text-align:right;"> 46.38 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Eugènia </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1778.33 </td>
   <td style="text-align:right;"> 3539.36 </td>
   <td style="text-align:right;"> 37.17 </td>
   <td style="text-align:right;"> 49.09 </td>
   <td style="text-align:right;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 186.97 </td>
   <td style="text-align:right;"> 155.61 </td>
   <td style="text-align:right;"> 22.40 </td>
   <td style="text-align:right;"> 32.16 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 186.47 </td>
   <td style="text-align:right;"> 156.93 </td>
   <td style="text-align:right;"> 22.99 </td>
   <td style="text-align:right;"> 32.84 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 227.61 </td>
   <td style="text-align:right;"> 194.13 </td>
   <td style="text-align:right;"> 25.08 </td>
   <td style="text-align:right;"> 34.56 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 252.82 </td>
   <td style="text-align:right;"> 524.11 </td>
   <td style="text-align:right;"> 27.76 </td>
   <td style="text-align:right;"> 36.50 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 233.22 </td>
   <td style="text-align:right;"> 523.63 </td>
   <td style="text-align:right;"> 29.50 </td>
   <td style="text-align:right;"> 38.08 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 873.96 </td>
   <td style="text-align:right;"> 2394.95 </td>
   <td style="text-align:right;"> 29.96 </td>
   <td style="text-align:right;"> 38.74 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1004.64 </td>
   <td style="text-align:right;"> 2522.56 </td>
   <td style="text-align:right;"> 32.48 </td>
   <td style="text-align:right;"> 41.51 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa Margalida </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1015.28 </td>
   <td style="text-align:right;"> 2555.07 </td>
   <td style="text-align:right;"> 35.01 </td>
   <td style="text-align:right;"> 43.56 </td>
   <td style="text-align:right;"> 405 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 402.29 </td>
   <td style="text-align:right;"> 565.66 </td>
   <td style="text-align:right;"> 22.95 </td>
   <td style="text-align:right;"> 40.90 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 423.33 </td>
   <td style="text-align:right;"> 649.19 </td>
   <td style="text-align:right;"> 23.43 </td>
   <td style="text-align:right;"> 41.36 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 344.58 </td>
   <td style="text-align:right;"> 171.73 </td>
   <td style="text-align:right;"> 25.19 </td>
   <td style="text-align:right;"> 42.99 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 568.95 </td>
   <td style="text-align:right;"> 1105.32 </td>
   <td style="text-align:right;"> 27.33 </td>
   <td style="text-align:right;"> 44.74 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 467.45 </td>
   <td style="text-align:right;"> 569.92 </td>
   <td style="text-align:right;"> 28.86 </td>
   <td style="text-align:right;"> 46.57 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 790.45 </td>
   <td style="text-align:right;"> 1975.04 </td>
   <td style="text-align:right;"> 29.29 </td>
   <td style="text-align:right;"> 46.91 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 887.50 </td>
   <td style="text-align:right;"> 2005.12 </td>
   <td style="text-align:right;"> 31.38 </td>
   <td style="text-align:right;"> 48.48 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santa María del Camí </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 872.24 </td>
   <td style="text-align:right;"> 1956.01 </td>
   <td style="text-align:right;"> 34.14 </td>
   <td style="text-align:right;"> 50.49 </td>
   <td style="text-align:right;"> 21 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 338.40 </td>
   <td style="text-align:right;"> 541.52 </td>
   <td style="text-align:right;"> 20.16 </td>
   <td style="text-align:right;"> 32.83 </td>
   <td style="text-align:right;"> 661 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 308.97 </td>
   <td style="text-align:right;"> 326.87 </td>
   <td style="text-align:right;"> 20.56 </td>
   <td style="text-align:right;"> 33.33 </td>
   <td style="text-align:right;"> 661 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 327.92 </td>
   <td style="text-align:right;"> 373.01 </td>
   <td style="text-align:right;"> 23.15 </td>
   <td style="text-align:right;"> 36.51 </td>
   <td style="text-align:right;"> 661 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 326.24 </td>
   <td style="text-align:right;"> 461.96 </td>
   <td style="text-align:right;"> 26.27 </td>
   <td style="text-align:right;"> 39.83 </td>
   <td style="text-align:right;"> 661 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 356.51 </td>
   <td style="text-align:right;"> 395.27 </td>
   <td style="text-align:right;"> 28.25 </td>
   <td style="text-align:right;"> 42.01 </td>
   <td style="text-align:right;"> 661 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1258.44 </td>
   <td style="text-align:right;"> 2735.88 </td>
   <td style="text-align:right;"> 28.53 </td>
   <td style="text-align:right;"> 42.37 </td>
   <td style="text-align:right;"> 661 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1382.57 </td>
   <td style="text-align:right;"> 2904.28 </td>
   <td style="text-align:right;"> 31.51 </td>
   <td style="text-align:right;"> 46.89 </td>
   <td style="text-align:right;"> 660 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Santanyí </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1322.05 </td>
   <td style="text-align:right;"> 2840.52 </td>
   <td style="text-align:right;"> 34.81 </td>
   <td style="text-align:right;"> 50.19 </td>
   <td style="text-align:right;"> 660 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 256.49 </td>
   <td style="text-align:right;"> 187.73 </td>
   <td style="text-align:right;"> 19.82 </td>
   <td style="text-align:right;"> 38.80 </td>
   <td style="text-align:right;"> 169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 255.16 </td>
   <td style="text-align:right;"> 181.16 </td>
   <td style="text-align:right;"> 20.20 </td>
   <td style="text-align:right;"> 39.35 </td>
   <td style="text-align:right;"> 169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 298.62 </td>
   <td style="text-align:right;"> 230.54 </td>
   <td style="text-align:right;"> 21.75 </td>
   <td style="text-align:right;"> 40.85 </td>
   <td style="text-align:right;"> 169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 461.62 </td>
   <td style="text-align:right;"> 1304.33 </td>
   <td style="text-align:right;"> 23.77 </td>
   <td style="text-align:right;"> 42.47 </td>
   <td style="text-align:right;"> 169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 447.46 </td>
   <td style="text-align:right;"> 1315.00 </td>
   <td style="text-align:right;"> 25.14 </td>
   <td style="text-align:right;"> 43.82 </td>
   <td style="text-align:right;"> 169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 839.79 </td>
   <td style="text-align:right;"> 2239.51 </td>
   <td style="text-align:right;"> 25.47 </td>
   <td style="text-align:right;"> 44.35 </td>
   <td style="text-align:right;"> 169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 929.08 </td>
   <td style="text-align:right;"> 2353.00 </td>
   <td style="text-align:right;"> 27.12 </td>
   <td style="text-align:right;"> 46.03 </td>
   <td style="text-align:right;"> 168 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Selva </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 973.11 </td>
   <td style="text-align:right;"> 2427.13 </td>
   <td style="text-align:right;"> 29.21 </td>
   <td style="text-align:right;"> 47.57 </td>
   <td style="text-align:right;"> 169 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 259.61 </td>
   <td style="text-align:right;"> 169.60 </td>
   <td style="text-align:right;"> 22.39 </td>
   <td style="text-align:right;"> 25.56 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 266.28 </td>
   <td style="text-align:right;"> 175.01 </td>
   <td style="text-align:right;"> 23.09 </td>
   <td style="text-align:right;"> 25.99 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 312.83 </td>
   <td style="text-align:right;"> 199.14 </td>
   <td style="text-align:right;"> 25.22 </td>
   <td style="text-align:right;"> 27.90 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 459.42 </td>
   <td style="text-align:right;"> 1210.47 </td>
   <td style="text-align:right;"> 28.27 </td>
   <td style="text-align:right;"> 29.88 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 433.05 </td>
   <td style="text-align:right;"> 1241.78 </td>
   <td style="text-align:right;"> 30.19 </td>
   <td style="text-align:right;"> 31.33 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1481.58 </td>
   <td style="text-align:right;"> 3067.94 </td>
   <td style="text-align:right;"> 30.85 </td>
   <td style="text-align:right;"> 32.03 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1182.74 </td>
   <td style="text-align:right;"> 2572.53 </td>
   <td style="text-align:right;"> 33.12 </td>
   <td style="text-align:right;"> 33.97 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sencelles </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1626.55 </td>
   <td style="text-align:right;"> 3105.56 </td>
   <td style="text-align:right;"> 36.12 </td>
   <td style="text-align:right;"> 35.96 </td>
   <td style="text-align:right;"> 67 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 192.75 </td>
   <td style="text-align:right;"> 120.75 </td>
   <td style="text-align:right;"> 25.43 </td>
   <td style="text-align:right;"> 44.66 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 193.63 </td>
   <td style="text-align:right;"> 110.89 </td>
   <td style="text-align:right;"> 26.15 </td>
   <td style="text-align:right;"> 45.51 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 251.59 </td>
   <td style="text-align:right;"> 173.50 </td>
   <td style="text-align:right;"> 29.04 </td>
   <td style="text-align:right;"> 47.72 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 251.38 </td>
   <td style="text-align:right;"> 169.13 </td>
   <td style="text-align:right;"> 32.15 </td>
   <td style="text-align:right;"> 49.35 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 216.20 </td>
   <td style="text-align:right;"> 162.75 </td>
   <td style="text-align:right;"> 34.43 </td>
   <td style="text-align:right;"> 50.78 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 747.99 </td>
   <td style="text-align:right;"> 2138.20 </td>
   <td style="text-align:right;"> 35.16 </td>
   <td style="text-align:right;"> 51.82 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1136.37 </td>
   <td style="text-align:right;"> 2725.01 </td>
   <td style="text-align:right;"> 38.47 </td>
   <td style="text-align:right;"> 53.99 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Ses Salines </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1129.72 </td>
   <td style="text-align:right;"> 2742.06 </td>
   <td style="text-align:right;"> 41.43 </td>
   <td style="text-align:right;"> 55.84 </td>
   <td style="text-align:right;"> 117 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 204.30 </td>
   <td style="text-align:right;"> 147.00 </td>
   <td style="text-align:right;"> 15.95 </td>
   <td style="text-align:right;"> 24.63 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 207.86 </td>
   <td style="text-align:right;"> 136.53 </td>
   <td style="text-align:right;"> 16.22 </td>
   <td style="text-align:right;"> 25.11 </td>
   <td style="text-align:right;"> 58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 234.82 </td>
   <td style="text-align:right;"> 145.71 </td>
   <td style="text-align:right;"> 17.88 </td>
   <td style="text-align:right;"> 26.35 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 245.79 </td>
   <td style="text-align:right;"> 185.68 </td>
   <td style="text-align:right;"> 20.07 </td>
   <td style="text-align:right;"> 27.58 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 231.40 </td>
   <td style="text-align:right;"> 152.68 </td>
   <td style="text-align:right;"> 21.02 </td>
   <td style="text-align:right;"> 28.59 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1066.12 </td>
   <td style="text-align:right;"> 2706.24 </td>
   <td style="text-align:right;"> 21.33 </td>
   <td style="text-align:right;"> 29.07 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1119.89 </td>
   <td style="text-align:right;"> 2727.77 </td>
   <td style="text-align:right;"> 23.25 </td>
   <td style="text-align:right;"> 30.52 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sineu </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 934.49 </td>
   <td style="text-align:right;"> 2451.51 </td>
   <td style="text-align:right;"> 25.51 </td>
   <td style="text-align:right;"> 31.73 </td>
   <td style="text-align:right;"> 57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 226.55 </td>
   <td style="text-align:right;"> 188.69 </td>
   <td style="text-align:right;"> 15.26 </td>
   <td style="text-align:right;"> 22.20 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 225.99 </td>
   <td style="text-align:right;"> 182.56 </td>
   <td style="text-align:right;"> 15.58 </td>
   <td style="text-align:right;"> 22.60 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 283.74 </td>
   <td style="text-align:right;"> 237.52 </td>
   <td style="text-align:right;"> 17.33 </td>
   <td style="text-align:right;"> 24.27 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 402.55 </td>
   <td style="text-align:right;"> 1107.96 </td>
   <td style="text-align:right;"> 19.40 </td>
   <td style="text-align:right;"> 25.88 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 386.83 </td>
   <td style="text-align:right;"> 1112.16 </td>
   <td style="text-align:right;"> 20.70 </td>
   <td style="text-align:right;"> 27.05 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 1189.89 </td>
   <td style="text-align:right;"> 2794.36 </td>
   <td style="text-align:right;"> 20.98 </td>
   <td style="text-align:right;"> 27.59 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 1617.18 </td>
   <td style="text-align:right;"> 3210.93 </td>
   <td style="text-align:right;"> 22.82 </td>
   <td style="text-align:right;"> 29.52 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Son Servera </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 1506.65 </td>
   <td style="text-align:right;"> 3087.20 </td>
   <td style="text-align:right;"> 24.86 </td>
   <td style="text-align:right;"> 31.02 </td>
   <td style="text-align:right;"> 159 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 275.26 </td>
   <td style="text-align:right;"> 324.08 </td>
   <td style="text-align:right;"> 41.83 </td>
   <td style="text-align:right;"> 49.55 </td>
   <td style="text-align:right;"> 314 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 283.61 </td>
   <td style="text-align:right;"> 319.95 </td>
   <td style="text-align:right;"> 43.11 </td>
   <td style="text-align:right;"> 50.80 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 336.66 </td>
   <td style="text-align:right;"> 407.44 </td>
   <td style="text-align:right;"> 47.29 </td>
   <td style="text-align:right;"> 53.39 </td>
   <td style="text-align:right;"> 312 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 322.70 </td>
   <td style="text-align:right;"> 354.24 </td>
   <td style="text-align:right;"> 51.91 </td>
   <td style="text-align:right;"> 55.89 </td>
   <td style="text-align:right;"> 312 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 263.06 </td>
   <td style="text-align:right;"> 262.38 </td>
   <td style="text-align:right;"> 55.36 </td>
   <td style="text-align:right;"> 58.34 </td>
   <td style="text-align:right;"> 312 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 710.30 </td>
   <td style="text-align:right;"> 1948.87 </td>
   <td style="text-align:right;"> 56.22 </td>
   <td style="text-align:right;"> 59.27 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 985.88 </td>
   <td style="text-align:right;"> 2306.90 </td>
   <td style="text-align:right;"> 60.88 </td>
   <td style="text-align:right;"> 62.55 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Sóller </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 990.88 </td>
   <td style="text-align:right;"> 2330.13 </td>
   <td style="text-align:right;"> 65.57 </td>
   <td style="text-align:right;"> 65.15 </td>
   <td style="text-align:right;"> 313 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 265.07 </td>
   <td style="text-align:right;"> 229.96 </td>
   <td style="text-align:right;"> 57.50 </td>
   <td style="text-align:right;"> 56.83 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 279.96 </td>
   <td style="text-align:right;"> 217.52 </td>
   <td style="text-align:right;"> 59.16 </td>
   <td style="text-align:right;"> 57.73 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 328.65 </td>
   <td style="text-align:right;"> 243.19 </td>
   <td style="text-align:right;"> 63.16 </td>
   <td style="text-align:right;"> 59.70 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 340.02 </td>
   <td style="text-align:right;"> 251.83 </td>
   <td style="text-align:right;"> 68.55 </td>
   <td style="text-align:right;"> 62.29 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 290.35 </td>
   <td style="text-align:right;"> 252.83 </td>
   <td style="text-align:right;"> 72.36 </td>
   <td style="text-align:right;"> 64.29 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 288.32 </td>
   <td style="text-align:right;"> 205.49 </td>
   <td style="text-align:right;"> 73.39 </td>
   <td style="text-align:right;"> 65.70 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 338.98 </td>
   <td style="text-align:right;"> 263.20 </td>
   <td style="text-align:right;"> 77.36 </td>
   <td style="text-align:right;"> 68.41 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valldemossa </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 346.77 </td>
   <td style="text-align:right;"> 260.55 </td>
   <td style="text-align:right;"> 81.46 </td>
   <td style="text-align:right;"> 70.71 </td>
   <td style="text-align:right;"> 56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2023-12-17 </td>
   <td style="text-align:right;"> 220.65 </td>
   <td style="text-align:right;"> 81.68 </td>
   <td style="text-align:right;"> 20.96 </td>
   <td style="text-align:right;"> 22.44 </td>
   <td style="text-align:right;"> 26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2024-03-23 </td>
   <td style="text-align:right;"> 227.62 </td>
   <td style="text-align:right;"> 117.94 </td>
   <td style="text-align:right;"> 21.35 </td>
   <td style="text-align:right;"> 22.92 </td>
   <td style="text-align:right;"> 26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2024-06-19 </td>
   <td style="text-align:right;"> 271.46 </td>
   <td style="text-align:right;"> 137.98 </td>
   <td style="text-align:right;"> 22.73 </td>
   <td style="text-align:right;"> 24.14 </td>
   <td style="text-align:right;"> 26 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2024-09-13 </td>
   <td style="text-align:right;"> 283.79 </td>
   <td style="text-align:right;"> 137.40 </td>
   <td style="text-align:right;"> 25.12 </td>
   <td style="text-align:right;"> 26.36 </td>
   <td style="text-align:right;"> 25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2024-12-14 </td>
   <td style="text-align:right;"> 270.36 </td>
   <td style="text-align:right;"> 127.35 </td>
   <td style="text-align:right;"> 26.44 </td>
   <td style="text-align:right;"> 27.30 </td>
   <td style="text-align:right;"> 25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2025-03-07 </td>
   <td style="text-align:right;"> 643.12 </td>
   <td style="text-align:right;"> 1784.13 </td>
   <td style="text-align:right;"> 26.84 </td>
   <td style="text-align:right;"> 28.08 </td>
   <td style="text-align:right;"> 25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2025-06-15 </td>
   <td style="text-align:right;"> 638.52 </td>
   <td style="text-align:right;"> 1746.61 </td>
   <td style="text-align:right;"> 28.16 </td>
   <td style="text-align:right;"> 29.71 </td>
   <td style="text-align:right;"> 25 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Vilafranc de Bonany </td>
   <td style="text-align:left;"> 2025-09-21 </td>
   <td style="text-align:right;"> 304.16 </td>
   <td style="text-align:right;"> 164.20 </td>
   <td style="text-align:right;"> 30.16 </td>
   <td style="text-align:right;"> 31.68 </td>
   <td style="text-align:right;"> 25 </td>
  </tr>
</tbody>
</table></div>

`````
:::
:::


## Pregunta 2 (**1punto**)

Consideremos las variables `price` y `number_of_reviews` de Pollença y Palma del periodo "2024-09-13", del fichero `listing_common0_select.RData`. 
Estudiad si estos datos se aproximan a una distribución normal gráficamente. Para ello, dibujad el histograma, la función "kernel-density" que aproxima la densidad y la densidad de la normal de media y varianza las de las muestras.


::: {.cell}

```{.r .cell-code}
# Filtrar datos para 2024-09-13
datos_p2 <- listings0 %>%
  filter(date == as.Date("2024-09-13"),
         neighbourhood_cleansed %in% c("Palma", "Pollença"),
         price > 50, price < 400)

# Gráfico para el Precio
ggplot(datos_p2, aes(x = price)) +
  geom_histogram(aes(y = after_stat(density)), bins = 30, fill = "lightblue", color = "black", alpha = 0.6) +
  geom_density(color = "red", linewidth = 1, aes(linetype = "Kernel")) +
  stat_function(fun = dnorm, 
                args = list(mean = mean(datos_p2$price, na.rm=TRUE), 
                            sd = sd(datos_p2$price, na.rm=TRUE)), 
                aes(color = "Normal Teórica"), linewidth = 1) +
  facet_wrap(~neighbourhood_cleansed) +
  labs(title = "Distribución del Precio (Palma vs Pollença)",
       subtitle = "Comparación Histograma vs Normal",
       y = "Densidad", x = "Precio") +
  theme_minimal()
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-9-1.png){width=672}
:::

```{.r .cell-code}
# Gráfico para Número de Reseñas
ggplot(datos_p2, aes(x = number_of_reviews)) +
  geom_histogram(aes(y = after_stat(density)), bins = 30, fill = "lightgreen", color = "black", alpha = 0.6) +
  geom_density(color = "red", linewidth = 1) +
  stat_function(fun = dnorm, 
                args = list(mean = mean(datos_p2$number_of_reviews, na.rm=TRUE), 
                            sd = sd(datos_p2$number_of_reviews, na.rm=TRUE)), 
                color = "blue", linewidth = 1) +
  facet_wrap(~neighbourhood_cleansed) +
  labs(title = "Distribución de Número de Reseñas", y = "Densidad", x = "Nº Reseñas") +
  theme_minimal()
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-9-2.png){width=672}
:::
:::


## Pregunta 3 (**1punto**)

Con los datos de `listings0` de todos los periodos, contrastar si la media del precio en Alcudia es igual a la de Palma **contra** que es mayor que en Palma para los precios mayores que 50 euros y menores de 400. Construid la hipótesis nula y alternativa, calculad el $p$-valor y el intervalo de confianza asociado al contraste.


::: {.cell}

```{.r .cell-code}
# Pregunta 3: Contraste de medias Alcúdia vs Palma

# 1. Unificamos correctamente los nombres de municipios
datos_p3 <- listings0 %>%
  mutate(
    neighbourhood_cleansed = str_squish(neighbourhood_cleansed),        # quita espacios
    neighbourhood_cleansed = str_to_sentence(neighbourhood_cleansed),   # uniformiza
    neighbourhood_cleansed = case_when(
      neighbourhood_cleansed %in% c("Alcudia", "Alcúdia") ~ "Alcúdia",
      neighbourhood_cleansed %in% c("Palma", "Palma de mallorca", "Palma De Mallorca", 
                                    "Palma de Mallorca") ~ "Palma",
      TRUE ~ neighbourhood_cleansed
    )
  ) %>%
  # 2. Filtrado de municipios y rango de precios
  filter(
    neighbourhood_cleansed %in% c("Alcúdia", "Palma"),
    price > 50, price < 400
  ) %>%
  mutate(neighbourhood_cleansed = droplevels(factor(neighbourhood_cleansed)))

# 3. Mostrar cuántos datos hay para cada municipio
cat("Número de observaciones por municipio después del filtrado:\n")
```

::: {.cell-output .cell-output-stdout}

```
Número de observaciones por municipio después del filtrado:
```


:::

```{.r .cell-code}
print(table(datos_p3$neighbourhood_cleansed))
```

::: {.cell-output .cell-output-stdout}

```

Alcúdia   Palma 
   6197    2881 
```


:::

```{.r .cell-code}
# 4. Verificar que realmente existen dos grupos
if (length(levels(datos_p3$neighbourhood_cleansed)) < 2) {

  cat("\n⚠️ No se puede realizar el t-test.\n")
  cat("Motivo: alguno de los municipios no tiene datos en el rango 50–400 euros.\n")

} else {

  # 5. Realizar contraste t-test
  t_test_p3 <- t.test(
    price ~ neighbourhood_cleansed,
    data = datos_p3,
    alternative = "greater"   # ¿Es Alcúdia más caro que Palma?
  )

  cat("\nResultado del t-test:\n")
  print(t_test_p3)

  # 6. Mostrar intervalo de confianza
  cat("\nIntervalo de confianza (unilateral, Alcúdia - Palma):\n")
  print(t_test_p3$conf.int)
}
```

::: {.cell-output .cell-output-stdout}

```

Resultado del t-test:

	Welch Two Sample t-test

data:  price by neighbourhood_cleansed
t = 5.1623, df = 5180.6, p-value = 1.265e-07
alternative hypothesis: true difference in means between group Alcúdia and group Palma is greater than 0
95 percent confidence interval:
 7.02432     Inf
sample estimates:
mean in group Alcúdia   mean in group Palma 
             193.9472              183.6373 


Intervalo de confianza (unilateral, Alcúdia - Palma):
[1] 7.02432     Inf
attr(,"conf.level")
[1] 0.95
```


:::
:::


## Pregunta 4 (**1punto**)

Con los datos de `listings0`, contrastar si las medias de los precios en Alcudia entre los periodos disponibles (Junio 2024 vs Septiembre 2024) son iguales contra que son menores en el primero.


::: {.cell}

```{.r .cell-code}
# Comparamos Junio 2024 vs Septiembre 2024 (fechas disponibles)
datos_p4 <- listings0 %>%
  # Aseguramos el nombre correcto usando str_detect para atrapar tildes
  filter(str_detect(neighbourhood_cleansed, "Alcudia|Alcúdia"),
         date %in% as.Date(c("2024-06-19", "2024-09-13")))

# T-test
t.test(price ~ date, data = datos_p4)
```

::: {.cell-output .cell-output-stdout}

```

	Welch Two Sample t-test

data:  price by date
t = -0.72325, df = 1228.1, p-value = 0.4697
alternative hypothesis: true difference in means between group 2024-06-19 and group 2024-09-13 is not equal to 0
95 percent confidence interval:
 -45.95988  21.20104
sample estimates:
mean in group 2024-06-19 mean in group 2024-09-13 
                272.2298                 284.6092 
```


:::

```{.r .cell-code}
# Boxplot
ggplot(datos_p4, aes(x = as.factor(date), y = price, fill = as.factor(date))) +
  geom_boxplot() +
  labs(title = "Precios en Alcudia: Junio 2024 vs Septiembre 2024", 
       x = "Fecha", y = "Precio", fill = "Fecha") +
  theme_bw()
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-11-1.png){width=672}
:::
:::


## Pregunta 5 (**1 punto**)

Comparar con un bopxlot de las valoraciones medias `review_scores_rating` para Alcudia, Palma, Calvià y Pollença.


::: {.cell}

```{.r .cell-code}
datos_p5 <- listings0 %>%
  filter(str_detect(neighbourhood_cleansed, "Alcudia|Alcúdia|Palma|Calvià|Pollença"))

ggplot(datos_p5, aes(x = neighbourhood_cleansed, y = review_scores_rating, fill = neighbourhood_cleansed)) +
  geom_boxplot(outlier.colour = "red", outlier.shape = 1) +
  labs(title = "Distribución de Valoraciones por Municipio",
       x = "Municipio", y = "Rating") +
  theme_classic() +
  theme(legend.position = "none")
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-12-1.png){width=672}
:::
:::


## Pregunta 6 (**1 punto**)

Calcular la proporción de apartamentos de la muestra "2024-09-13" con media de valoración `review_scores_rating` mayor que 4 en Alcudia y en Calvià son iguales contra que son distintas.


::: {.cell}

```{.r .cell-code}
datos_p6 <- listings0 %>%
  filter(date == as.Date("2024-09-13"),
         str_detect(neighbourhood_cleansed, "Alcudia|Alcúdia|Calvià")) %>%
  mutate(high_rating = ifelse(review_scores_rating > 4, 1, 0))

# Limpiamos nombres para la tabla
datos_p6 <- datos_p6 %>%
  mutate(neighbourhood_cleansed = str_replace(neighbourhood_cleansed, "Alcúdia", "Alcudia"))

tabla_prop6 <- table(datos_p6$neighbourhood_cleansed, datos_p6$high_rating)
print(tabla_prop6)
```

::: {.cell-output .cell-output-stdout}

```
         
            0   1
  Alcudia  53 818
  Calvià   11 159
```


:::

```{.r .cell-code}
prop.test(x = tabla_prop6[, "1"], n = rowSums(tabla_prop6))
```

::: {.cell-output .cell-output-stdout}

```

	2-sample test for equality of proportions with continuity correction

data:  tabla_prop6[, "1"] out of rowSums(tabla_prop6)
X-squared = 0.00028674, df = 1, p-value = 0.9865
alternative hypothesis: two.sided
95 percent confidence interval:
 -0.03990293  0.04761550
sample estimates:
   prop 1    prop 2 
0.9391504 0.9352941 
```


:::
:::


## Pregunta 7 (**1punto**)

Calcular la proporción de apartamentos de los periodos 2024-06-19 y 2024-09-13 con media de valoración `review_scores_rating` mayor que 4 en Palma y en Pollença.


::: {.cell}

```{.r .cell-code}
datos_p7 <- listings0 %>%
  filter(date %in% as.Date(c("2024-06-19", "2024-09-13")),
         neighbourhood_cleansed %in% c("Palma", "Pollença")) %>%
  mutate(high_rating = ifelse(review_scores_rating > 4, 1, 0))

tabla_prop7 <- table(datos_p7$neighbourhood_cleansed, datos_p7$high_rating)
prop.test(x = tabla_prop7[, "1"], n = rowSums(tabla_prop7))
```

::: {.cell-output .cell-output-stdout}

```

	1-sample proportions test with continuity correction

data:  tabla_prop7[, "1"] out of rowSums(tabla_prop7), null probability 0.5
X-squared = 1734.6, df = 1, p-value < 2.2e-16
alternative hypothesis: true p is not equal to 0.5
95 percent confidence interval:
 0.8974260 0.9199438
sample estimates:
        p 
0.9093014 
```


:::
:::


### Pregunta 7 (Contingencia)

Agrupa las variables `review_scores_rating` y `review_scores_location` en 5 categorías y testea independencia.


::: {.cell}

```{.r .cell-code}
# Tabla de contingencia
tabla_chi <- table(cut(listings0$review_scores_rating, 5),
                   cut(listings0$review_scores_location, 5))

print(tabla_chi)
```

::: {.cell-output .cell-output-stdout}

```
             
              (0.996,1.8] (1.8,2.6] (2.6,3.4] (3.4,4.2] (4.2,5]
  (0.996,1.8]          17         8         6        10       2
  (1.8,2.6]             8        11        12        30      34
  (2.6,3.4]             8        20        59       125     168
  (3.4,4.2]             0         0       105      1217    2905
  (4.2,5]               0         9        93      2313   57567
```


:::

```{.r .cell-code}
# Test Chi-Cuadrado
chi_test <- chisq.test(tabla_chi)
print(chi_test)
```

::: {.cell-output .cell-output-stdout}

```

	Pearson's Chi-squared test

data:  tabla_chi
X-squared = 28398, df = 16, p-value < 2.2e-16
```


:::

```{.r .cell-code}
# Coeficiente de Contingencia de Pearson
N_total <- sum(tabla_chi)
coef_contingencia <- sqrt(chi_test$statistic / (N_total + chi_test$statistic))

cat("Coeficiente de Contingencia:", coef_contingencia, "\n")
```

::: {.cell-output .cell-output-stdout}

```
Coeficiente de Contingencia: 0.55222 
```


:::
:::


## Pregunta 9 (**3 puntos**)

Construye un data set con las variables de puntuación y calcula la matriz de correlaciones.


::: {.cell}

```{.r .cell-code}
# Seleccionar variables
datos_corr <- listings0 %>%
  select(neighbourhood_cleansed, 
         review_scores_rating, 
         review_scores_cleanliness, 
         review_scores_location, 
         review_scores_value) %>%
  na.omit()

# Matriz de correlación
matriz_corr <- cor(datos_corr %>% select(-neighbourhood_cleansed))

# Gráfico de pares (GGally) - Usamos una muestra para agilizar
ggpairs(datos_corr %>% sample_n(min(1000, n())), 
        aes(color = neighbourhood_cleansed, alpha = 0.5),
        columns = 2:5) +
        labs(title = "Relaciones entre Puntuaciones")
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-16-1.png){width=672}
:::

```{.r .cell-code}
# Gráfico de matriz (Corrplot)
corrplot(matriz_corr, method = "ellipse", type = "upper", 
         tl.col = "black", addCoef.col = "black",
         title = "Matriz de Correlaciones")
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-16-2.png){width=672}
:::
:::


## Pregunta 9 (**2 puntos**)

**Ley de Zipf**. Análisis de la longitud de los comentarios y descripciones.

**1. Análisis para las Reseñas (Reviews)**


::: {.cell}

```{.r .cell-code}
# Contamos palabras
length_rewiews = stringr::str_count(reviews$comments, "\\w+")

barplot(table(length_rewiews), 
        main = "Distribución Longitud Reviews", xlab = "Nº Palabras")
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-17-1.png){width=672}
:::

```{.r .cell-code}
# Preparación para Zipf
aux = table(length_rewiews)

tbl_rev = tibble(
  L = as.numeric(names(aux)),
  Freq = as.numeric(aux)
) %>%
  arrange(desc(Freq)) %>%  # ORDENAR por frecuencia descendente
  mutate(
    Rank = row_number(),
    Log_Freq = log(Freq),
    Log_Rank = log(Rank)
  )

# Filtrar colas (Rank > 10 y < 1000)
tbl2_rev = tbl_rev %>% filter(Rank > 10, Rank < 1000)

# Modelo Log-Log (Zipf)
sol3_rev = lm(Log_Freq ~ Log_Rank, data = tbl2_rev)
summary(sol3_rev)
```

::: {.cell-output .cell-output-stdout}

```

Call:
lm(formula = Log_Freq ~ Log_Rank, data = tbl2_rev)

Residuals:
    Min      1Q  Median      3Q     Max 
-4.2549 -0.3422 -0.0502  0.4045  1.3674 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept) 20.27720    0.20998   96.57   <2e-16 ***
Log_Rank    -3.08397    0.03785  -81.48   <2e-16 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 0.7782 on 597 degrees of freedom
Multiple R-squared:  0.9175,	Adjusted R-squared:  0.9174 
F-statistic:  6639 on 1 and 597 DF,  p-value: < 2.2e-16
```


:::

```{.r .cell-code}
# Gráfico Log-Log Reviews
ggplot(tbl2_rev, aes(x = Log_Rank, y = Log_Freq)) +
  geom_point(alpha = 0.5, color = "red") +
  geom_smooth(method = "lm", color = "black") +
  labs(title = "Ley de Zipf: Longitud de Reviews",
       subtitle = paste("Pendiente:", round(coef(sol3_rev)[2], 3), 
                        "| R2:", round(summary(sol3_rev)$r.squared, 3)),
       x = "log(Rango)", y = "log(Frecuencia)") +
  theme_minimal()
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-17-2.png){width=672}
:::
:::


**2. Análisis para las Descripciones (Listings)**


::: {.cell}

```{.r .cell-code}
# Contar palabras en descripciones
length_description = stringr::str_count(listings0$description, "\\w+")
length_description = na.omit(length_description)

barplot(table(length_description), main = "Distribución Longitud Descripciones")
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-18-1.png){width=672}
:::

```{.r .cell-code}
# Preparación Zipf
aux_desc = table(length_description)

tbl_desc = tibble(
  L = as.numeric(names(aux_desc)),
  Freq = as.numeric(aux_desc)
) %>%
  arrange(desc(Freq)) %>%  # ORDENAR por frecuencia
  mutate(
    Rank = row_number(),
    Log_Freq = log(Freq),
    Log_Rank = log(Rank)
  )

# Filtrar
tbl2_desc = tbl_desc %>% filter(Rank > 10, Rank < 1000)

# Modelo Log-Log
sol3_desc = lm(Log_Freq ~ Log_Rank, data = tbl2_desc)
summary(sol3_desc)
```

::: {.cell-output .cell-output-stdout}

```

Call:
lm(formula = Log_Freq ~ Log_Rank, data = tbl2_desc)

Residuals:
    Min      1Q  Median      3Q     Max 
-2.6301 -0.9624 -0.1638  1.0706  1.7943 

Coefficients:
            Estimate Std. Error t value Pr(>|t|)    
(Intercept)  15.6074     0.6660   23.43   <2e-16 ***
Log_Rank     -2.6186     0.1577  -16.60   <2e-16 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 1.175 on 131 degrees of freedom
Multiple R-squared:  0.6778,	Adjusted R-squared:  0.6754 
F-statistic: 275.6 on 1 and 131 DF,  p-value: < 2.2e-16
```


:::

```{.r .cell-code}
# Gráfico Log-Log Descriptions
ggplot(tbl2_desc, aes(x = Log_Rank, y = Log_Freq)) +
  geom_point(alpha = 0.5, color = "blue") +
  geom_smooth(method = "lm", color = "black") +
  labs(title = "Ley de Zipf: Longitud de Descripciones",
       subtitle = paste("Pendiente:", round(coef(sol3_desc)[2], 3), 
                        "| R2:", round(summary(sol3_desc)$r.squared, 3)),
       x = "log(Rango)", y = "log(Frecuencia)") +
  theme_minimal()
```

::: {.cell-output-display}
![](ENUNCIADO_taller_EVALUABLE_ABB_files/figure-html/unnamed-chunk-18-2.png){width=672}
:::
:::
