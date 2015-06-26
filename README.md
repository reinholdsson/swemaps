## swemaps

**Swedish map data for ggplot and leaflet in R**
  
```{r}
devtools::install_github('reinholdsson/swemaps')
```

### Datasets

- `map_ln`: regions polygon data
- `map_kn`: municipalities polygon data
- `cent_ln`: regions centroids data
- `cent_kn`: municipalities centroids data

### Examples

#### ggplot2

```{r}
library(ggplot2)
library(swemaps)
library(rkolada)  # devtools::install_github("reinholdsson/rkolada")

# function to merge kolada data with map data from swemaps
prepare_map_data <- function(x) {
  x$knkod <- x$municipality.id
  data <- merge(map_kn, x, by = 'knkod')
  data[order(data$order),]  # make sure it's sorted by "order" column
}

# rkolada conn
a <- rkolada::rkolada()

# Get data from Kolada
x <- a$values('N00941', year = 2010)  # make sure "kpi.municipality_type == 'K'"
x <- prepare_map_data(x)

# Plot it!
ggplot(x, aes_string('ggplot_long', 'ggplot_lat', group = 'knkod', fill = 'value')) +
  geom_polygon() +
  coord_equal()
```

![](/img/example_1.png?raw=true)

```{r}
x <- a$values('N00941', c('1440', '0604', '2505', '2084'), year = 2010, all.cols = T)
x <- prepare_map_data(x)
ggplot(x, aes_string('ggplot_long', 'ggplot_lat', group = 'knkod', fill = 'value')) +
  geom_polygon() +
  coord_equal() +
  facet_wrap(~municipality.title, scales = 'free', ncol = 2) +
  theme_bw()
```

![](/img/example_2.png?raw=true)

#### leaflet

https://rstudio.github.io/leaflet/
  
```{r}
library(leaflet)  # devtools::install_github("rstudio/leaflet")

x <- map_kn

m <- leaflet() %>% addTiles()

for (kn in unique(x$knkod)) {
  i <- x[x$knkod == kn,]
  m <- m %>% addPolygons(i$leaflet_long, i$leaflet_lat, color = 'blue', weight = 1)
}

m  # plot!
```

![](/img/example_3.png?raw=true)

with coloured areas:

```{r}
library(plotrix)

x <- a$values('N01963', year = 2010)
x <- subset(x, gender == 'T')
x <- prepare_map_data(x)
x$color <- substring(color.scale(x$value, c(1,1,0), c(0,1,1), 0), 1, 7)

m <- leaflet() %>% addTiles()
for (kn in unique(x$knkod)) {
  i <- x[x$knkod == kn,]
  m <- m %>% addPolygons(i$leaflet_long, i$leaflet_lat, color = i$color[[1]], weight = 1)
}
m
```

![](/img/example_4.png?raw=true)

### Source

arcgis (Kommungränser_SCB_07, Länsgränser_SCB_07), SCB
