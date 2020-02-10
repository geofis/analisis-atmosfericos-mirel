Análisis de datos atmosféricos
================

## Instalar paquete S5Processor

``` r
#devtools::install_github("MBalthasar/S5Processor")
```

## Cargar paquetes

``` r
library(S5Processor)
library(raster)
```

    ## Loading required package: sp

``` r
library(sf)
```

    ## Linking to GEOS 3.6.2, GDAL 2.2.3, PROJ 4.9.3

``` r
library(readr)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.2.1     ✔ purrr   0.3.2
    ## ✔ tibble  2.1.3     ✔ dplyr   0.8.3
    ## ✔ tidyr   1.0.0     ✔ stringr 1.4.0
    ## ✔ ggplot2 3.2.1     ✔ forcats 0.4.0

    ## ── Conflicts ────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ tidyr::extract() masks raster::extract()
    ## ✖ dplyr::filter()  masks stats::filter()
    ## ✖ dplyr::lag()     masks stats::lag()
    ## ✖ dplyr::select()  masks raster::select()

Cargar
datos

``` r
f <- 'S5P_NRTI_L2__NO2____20191209T173610_20191209T174110_11170_01_010302_20191209T182308.nc'
nc <- ncdf4::nc_open(f)
vars <- attributes(nc$var)$names
p <- 6
p
ncr <- S5P_process(input = f, product = p)
ncr
```

## Exportar

``` r
writeRaster(ncr, paste0(gsub('.nc', '', f), '_', gsub('/','_', vars[p]), '.tif'), overwrite=T)
```

## Instalar sentinelsat (API de Sentinel)

``` bash
git clone https://github.com/sentinelsat/sentinelsat.git
sudo apt install python-pip
pip install sentinelsat
sudo find / -type f -iname '*sentinelsat*'
/home/mirel/.local/bin/sentinelsat -u s5pguest -p s5pguest -g /home/mirel/Documents/analisis-atmosfericos-mirel/descargas/rd.geojson --sentinel 5 -s 20181018 -e 20191018 --url "https://s5phub.copernicus.eu/dhus" --producttype L2__NO2___
#Footprins y metadatos:
/home/mirel/.local/bin/sentinelsat -u s5pguest -p s5pguest -d -g /home/mirel/Documents/analisis-atmosfericos-mirel/descargas/rd.geojson --sentinel 5 -s 20181018 -e 20181019 --url "https://s5phub.copernicus.eu/dhus" --producttype L2__NO2___ --footprints
#Para descargar, agrega flag -d, ejemplo:
/home/mirel/.local/bin/sentinelsat -u s5pguest -p s5pguest -d -g /home/mirel/Documents/analisis-atmosfericos-mirel/descargas/rd.geojson --sentinel 5 -s 20181018 -e 20181019 --url "https://s5phub.copernicus.eu/dhus" --producttype L2__NO2___
```

# Revisar y descargar pendientes (ejecutar al finalizar la descarga por fecha)

``` r
fp <- st_read('descargas/footprints-20181018-20191018.geojson')
nombrestodos <- gsub('\\.nc', '', fp$filename)
descargados_df <- read_delim('lista-descargas-a-20200210.txt', delim = ' ', col_names = F)
#El archivo "lista-descargas-a-20200210.txt" se genera al ejecutar, en la terminal,
#ls -al > lista-descargas-a-20200210.txt
#dentro de la carpeta donde se encuentren todas las imágenes
descargados <- gsub('\\.zip$', '', grep('^S5P.*', descargados_df$X9, value = T))
match(nombrestodos, descargados)
#N pendientes
length(which(is.na(match(nombrestodos, descargados))))
#Indice pendientes
uuidpendientes <- fp %>%
  slice(which(is.na(match(nombrestodos, descargados)))) %>%
  pull(uuid) %>% as.character()
sapply(
  uuidpendientes,
  function(x) {
    system(
      paste0(
        '/home/mirel/.local/bin/sentinelsat ',
        '--uuid ', x,
        ' -u s5pguest -p s5pguest -d ',
        '-g /home/mirel/Documents/analisis-atmosfericos-mirel/descargas/rd.geojson ',
        '--sentinel 5 ',
        '--url "https://s5phub.copernicus.eu/dhus"'
      )
    )
  }
)
```
