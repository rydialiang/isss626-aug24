---
title: "5 2nd Order Spatial Point Patterns Analysis Methods"
author: "Liang Xiuhao Rydia"
date: "Aug 31, 2024"
date-modified: "last-modified"
execute:
  eval: true
  echo: true
  warning: false
  freeze: true
format: 
  html:
    code-fold: true
    code-summary: "Show the code"
---


![](images/clipboard-316435035.png)

## **5.1 Overview**

Spatial Point Pattern Analysis is the evaluation of the pattern or distribution, of a set of points on a surface. The point can be location of:

-   events such as crime, traffic accident and disease onset, or

-   business services (coffee and fastfood outlets) or facilities such as childcare and eldercare.

Using appropriate functions of [spatstat](https://cran.r-project.org/web/packages/spatstat/), this hands-on exercise aims to discover the spatial point processes of childecare centres in Singapore.

The specific questions we would like to answer are as follows:

-   are the childcare centres in Singapore randomly distributed throughout the country?

-   if the answer is not, then the next logical question is where are the locations with higher concentration of childcare centres?

## **5.2 The data**

To provide answers to the questions above, three data sets will be used. They are:

-   `CHILDCARE`, a point feature data providing both location and attribute information of childcare centres. It was downloaded from Data.gov.sg and is in geojson format.

-   `MP14_SUBZONE_WEB_PL`, a polygon feature data providing information of URA 2014 Master Plan Planning Subzone boundary data. It is in ESRI shapefile format. This data set was also downloaded from Data.gov.sg.

-   `CostalOutline`, a polygon feature data showing the national boundary of Singapore. It is provided by SLA and is in ESRI shapefile format.

## **5.3 Installing and Loading the R packages**

In this hands-on exercise, five R packages will be used, they are:

-   [**sf**](https://r-spatial.github.io/sf/), a relatively new R package specially designed to import, manage and process vector-based geospatial data in R.

-   [**spatstat**](https://spatstat.org/), which has a wide range of useful functions for point pattern analysis. In this hands-on exercise, it will be used to perform 1st- and 2nd-order spatial point patterns analysis and derive kernel density estimation (KDE) layer.

-   [**raster**](https://cran.r-project.org/web/packages/raster/) which reads, writes, manipulates, analyses and model of gridded spatial data (i.e. raster). In this hands-on exercise, it will be used to convert image output generate by spatstat into raster format.

-   [**maptools**](https://cran.r-project.org/web/packages/maptools/index.html) which provides a set of tools for manipulating geographic data. In this hands-on exercise, we mainly use it to convert *Spatial* objects into *ppp* format of **spatstat**.

-   [**tmap**](https://cran.r-project.org/web/packages/tmap/index.html) which provides functions for plotting cartographic quality static point patterns maps or interactive maps by using [leaflet](https://leafletjs.com/) API.

Use the code chunk below to install and launch the five R packages.


::: {.cell}

```{.r .cell-code}
pacman::p_load(sf, raster, spatstat, tmap, tidyverse)
```
:::


## **5.4 Spatial Data Wrangling**

### **5.4.1 Importing the spatial data**

In this section, [*st_read()*](https://r-spatial.github.io/sf/reference/st_read.html) of **sf** package will be used to import these three geospatial data sets into R.


::: {.cell}

```{.r .cell-code}
childcare_sf <- st_read("data/child-care-services-geojson.geojson") %>%
  st_transform(crs = 3414)
```

::: {.cell-output .cell-output-stdout}

```
Reading layer `child-care-services-geojson' from data source 
  `C:\rydialiang\isss626-aug24\Hands-on Exercise\Hands-on_Ex02b\data\child-care-services-geojson.geojson' 
  using driver `GeoJSON'
Simple feature collection with 1545 features and 2 fields
Geometry type: POINT
Dimension:     XYZ
Bounding box:  xmin: 103.6824 ymin: 1.248403 xmax: 103.9897 ymax: 1.462134
z_range:       zmin: 0 zmax: 0
Geodetic CRS:  WGS 84
```


:::

```{.r .cell-code}
sg_sf <- st_read(dsn = "data", layer="CostalOutline")
```

::: {.cell-output .cell-output-stdout}

```
Reading layer `CostalOutline' from data source 
  `C:\rydialiang\isss626-aug24\Hands-on Exercise\Hands-on_Ex02b\data' 
  using driver `ESRI Shapefile'
Simple feature collection with 60 features and 4 fields
Geometry type: POLYGON
Dimension:     XY
Bounding box:  xmin: 2663.926 ymin: 16357.98 xmax: 56047.79 ymax: 50244.03
Projected CRS: SVY21
```


:::

```{.r .cell-code}
mpsz_sf <- st_read(dsn = "data", 
                layer = "MP14_SUBZONE_WEB_PL")
```

::: {.cell-output .cell-output-stdout}

```
Reading layer `MP14_SUBZONE_WEB_PL' from data source 
  `C:\rydialiang\isss626-aug24\Hands-on Exercise\Hands-on_Ex02b\data' 
  using driver `ESRI Shapefile'
Simple feature collection with 323 features and 15 fields
Geometry type: MULTIPOLYGON
Dimension:     XY
Bounding box:  xmin: 2667.538 ymin: 15748.72 xmax: 56396.44 ymax: 50256.33
Projected CRS: SVY21
```


:::
:::


Before we can use these data for analysis, it is important for us to ensure that they are projected in same projection system.

> DIY: Using the appropriate **sf** function you learned in Hands-on Exercise 2, retrieve the referencing system information of these geospatial data.

Notice that except `childcare_sf`, both `mpsz_sf` and `sg_sf` do not have proper crs information.

> DIY: Using the method you learned in Lesson 2, assign the correct crs to mpsz_sf and sg_sf simple feature data frames.

> DIY: If necessary, changing the referencing system to Singapore national projected coordinate system.

### **5.4.2 Mapping the geospatial data sets**

After checking the referencing system of each geospatial data data frame, it is also useful for us to plot a map to show their spatial patterns.

> DIY: Using the mapping methods you learned in Hands-on Exercise 3, prepare a map as shown below.

![](images/clipboard-201045715.png)


::: {.cell}

```{.r .cell-code}
plot(st_geometry(mpsz_sf),
     col='lightgrey')
plot(childcare_sf,
     add=T,
     col='black',
     fill='black',
     pch=22,
     cex=.1)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-3-1.png){width=672}
:::
:::


Notice that all the geospatial layers are within the same map extend. This shows that their referencing system and coordinate values are referred to similar spatial context. This is very important in any geospatial analysis.

Alternatively, we can also prepare a pin map by using the code chunk below.


::: {.cell}

```{.r .cell-code}
tmap_mode('view')
tm_shape(childcare_sf)+
  tm_dots()
```

::: {.cell-output-display}

```{=html}
<div class="leaflet html-widget html-fill-item" id="htmlwidget-874824cb71372ebf465d" style="width:100%;height:464px;"></div>
```

:::
:::

::: {.cell}

```{.r .cell-code}
tmap_mode('plot')
```
:::


Although simple feature data frame is gaining popularity again sp’s Spatial\* classes, there are, however, many geospatial analysis packages require the input geospatial data in sp’s Spatial\* classes. In this section, you will learn how to convert simple feature data frame to sp’s Spatial\* class.

### **5.5.1 Converting from sf format into spatstat’s ppp format**

Now, we will use *as.ppp()* function of **spatstat** to convert the spatial data into **spatstat**’s ***ppp*** object format.


::: {.cell}

```{.r .cell-code}
childcare_ppp <- as.ppp(childcare_sf)
childcare_ppp
```

::: {.cell-output .cell-output-stdout}

```
Marked planar point pattern: 1545 points
marks are of storage type  'character'
window: rectangle = [11203.01, 45404.24] x [25667.6, 49300.88] units
```


:::
:::


Plot childcare_ppp and examine the difference.


::: {.cell}

```{.r .cell-code}
plot(childcare_ppp)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-7-1.png){width=672}
:::
:::


You can take a quick look at the summary statistics of the newly created ppp object by using the code chunk below.


::: {.cell}

```{.r .cell-code}
summary(childcare_ppp)
```

::: {.cell-output .cell-output-stdout}

```
Marked planar point pattern:  1545 points
Average intensity 1.91145e-06 points per square unit

Coordinates are given to 11 decimal places

marks are of type 'character'
Summary:
   Length     Class      Mode 
     1545 character character 

Window: rectangle = [11203.01, 45404.24] x [25667.6, 49300.88] units
                    (34200 x 23630 units)
Window area = 808287000 square units
```


:::
:::


Checking for duplicate:


::: {.cell}

```{.r .cell-code}
childcare_ppp_jit <- rjitter(childcare_ppp, 
                             retry=TRUE, 
                             nsim=1, 
                             drop=TRUE)

any(duplicated(childcare_ppp_jit))
```

::: {.cell-output .cell-output-stdout}

```
[1] FALSE
```


:::
:::


### **5.5.3 Creating *owin* object**

When analysing spatial point patterns, it is a good practice to confine the analysis with a geographical area like Singapore boundary. In **spatstat**, an object called ***owin*** is specially designed to represent this polygonal region.

The code chunk below is used to covert *sg* SpatialPolygon object into owin object of **spatstat**.


::: {.cell}

```{.r .cell-code}
sg_owin <- as.owin(sg_sf)
```
:::


The ouput object can be displayed by using *plot()* function


::: {.cell}

```{.r .cell-code}
plot(sg_owin)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-11-1.png){width=672}
:::
:::

::: {.cell}

```{.r .cell-code}
summary(sg_owin)
```

::: {.cell-output .cell-output-stdout}

```
Window: polygonal boundary
50 separate polygons (1 hole)
                 vertices         area relative.area
polygon 1 (hole)       30     -7081.18     -9.76e-06
polygon 2              55     82537.90      1.14e-04
polygon 3              90    415092.00      5.72e-04
polygon 4              49     16698.60      2.30e-05
polygon 5              38     24249.20      3.34e-05
polygon 6             976  23344700.00      3.22e-02
polygon 7             721   1927950.00      2.66e-03
polygon 8            1992   9992170.00      1.38e-02
polygon 9             330   1118960.00      1.54e-03
polygon 10            175    925904.00      1.28e-03
polygon 11            115    928394.00      1.28e-03
polygon 12             24      6352.39      8.76e-06
polygon 13            190    202489.00      2.79e-04
polygon 14             37     10170.50      1.40e-05
polygon 15             25     16622.70      2.29e-05
polygon 16             10      2145.07      2.96e-06
polygon 17             66     16184.10      2.23e-05
polygon 18           5195 636837000.00      8.78e-01
polygon 19             76    312332.00      4.31e-04
polygon 20            627  31891300.00      4.40e-02
polygon 21             20     32842.00      4.53e-05
polygon 22             42     55831.70      7.70e-05
polygon 23             67   1313540.00      1.81e-03
polygon 24            734   4690930.00      6.47e-03
polygon 25             16      3194.60      4.40e-06
polygon 26             15      4872.96      6.72e-06
polygon 27             15      4464.20      6.15e-06
polygon 28             14      5466.74      7.54e-06
polygon 29             37      5261.94      7.25e-06
polygon 30            111    662927.00      9.14e-04
polygon 31             69     56313.40      7.76e-05
polygon 32            143    145139.00      2.00e-04
polygon 33            397   2488210.00      3.43e-03
polygon 34             90    115991.00      1.60e-04
polygon 35             98     62682.90      8.64e-05
polygon 36            165    338736.00      4.67e-04
polygon 37            130     94046.50      1.30e-04
polygon 38             93    430642.00      5.94e-04
polygon 39             16      2010.46      2.77e-06
polygon 40            415   3253840.00      4.49e-03
polygon 41             30     10838.20      1.49e-05
polygon 42             53     34400.30      4.74e-05
polygon 43             26      8347.58      1.15e-05
polygon 44             74     58223.40      8.03e-05
polygon 45            327   2169210.00      2.99e-03
polygon 46            177    467446.00      6.44e-04
polygon 47             46    699702.00      9.65e-04
polygon 48              6     16841.00      2.32e-05
polygon 49             13     70087.30      9.66e-05
polygon 50              4      9459.63      1.30e-05
enclosing rectangle: [2663.93, 56047.79] x [16357.98, 50244.03] units
                     (53380 x 33890 units)
Window area = 725376000 square units
Fraction of frame area: 0.401
```


:::
:::


### **5.5.4 Combining point events object and owin object**

In this last step of geospatial data wrangling, we will extract childcare events that are located within Singapore by using the code chunk below.


::: {.cell}

```{.r .cell-code}
childcareSG_ppp = childcare_ppp[sg_owin]
```
:::


The output object combined both the point and polygon feature in one ppp object class as shown below.


::: {.cell}

```{.r .cell-code}
summary(childcareSG_ppp)
```

::: {.cell-output .cell-output-stdout}

```
Marked planar point pattern:  1545 points
Average intensity 2.129929e-06 points per square unit

Coordinates are given to 11 decimal places

marks are of type 'character'
Summary:
   Length     Class      Mode 
     1545 character character 

Window: polygonal boundary
50 separate polygons (1 hole)
                 vertices         area relative.area
polygon 1 (hole)       30     -7081.18     -9.76e-06
polygon 2              55     82537.90      1.14e-04
polygon 3              90    415092.00      5.72e-04
polygon 4              49     16698.60      2.30e-05
polygon 5              38     24249.20      3.34e-05
polygon 6             976  23344700.00      3.22e-02
polygon 7             721   1927950.00      2.66e-03
polygon 8            1992   9992170.00      1.38e-02
polygon 9             330   1118960.00      1.54e-03
polygon 10            175    925904.00      1.28e-03
polygon 11            115    928394.00      1.28e-03
polygon 12             24      6352.39      8.76e-06
polygon 13            190    202489.00      2.79e-04
polygon 14             37     10170.50      1.40e-05
polygon 15             25     16622.70      2.29e-05
polygon 16             10      2145.07      2.96e-06
polygon 17             66     16184.10      2.23e-05
polygon 18           5195 636837000.00      8.78e-01
polygon 19             76    312332.00      4.31e-04
polygon 20            627  31891300.00      4.40e-02
polygon 21             20     32842.00      4.53e-05
polygon 22             42     55831.70      7.70e-05
polygon 23             67   1313540.00      1.81e-03
polygon 24            734   4690930.00      6.47e-03
polygon 25             16      3194.60      4.40e-06
polygon 26             15      4872.96      6.72e-06
polygon 27             15      4464.20      6.15e-06
polygon 28             14      5466.74      7.54e-06
polygon 29             37      5261.94      7.25e-06
polygon 30            111    662927.00      9.14e-04
polygon 31             69     56313.40      7.76e-05
polygon 32            143    145139.00      2.00e-04
polygon 33            397   2488210.00      3.43e-03
polygon 34             90    115991.00      1.60e-04
polygon 35             98     62682.90      8.64e-05
polygon 36            165    338736.00      4.67e-04
polygon 37            130     94046.50      1.30e-04
polygon 38             93    430642.00      5.94e-04
polygon 39             16      2010.46      2.77e-06
polygon 40            415   3253840.00      4.49e-03
polygon 41             30     10838.20      1.49e-05
polygon 42             53     34400.30      4.74e-05
polygon 43             26      8347.58      1.15e-05
polygon 44             74     58223.40      8.03e-05
polygon 45            327   2169210.00      2.99e-03
polygon 46            177    467446.00      6.44e-04
polygon 47             46    699702.00      9.65e-04
polygon 48              6     16841.00      2.32e-05
polygon 49             13     70087.30      9.66e-05
polygon 50              4      9459.63      1.30e-05
enclosing rectangle: [2663.93, 56047.79] x [16357.98, 50244.03] units
                     (53380 x 33890 units)
Window area = 725376000 square units
Fraction of frame area: 0.401
```


:::
:::


#### 5.5.4.1 Extracting study area

The code chunk below will be used to extract the target planning areas


::: {.cell}

```{.r .cell-code}
pg <- mpsz_sf %>%
  filter(PLN_AREA_N == "PUNGGOL")
tm <- mpsz_sf %>%
  filter(PLN_AREA_N == "TAMPINES")
ck <- mpsz_sf %>%
  filter(PLN_AREA_N == "CHOA CHU KANG")
jw <- mpsz_sf %>%
  filter(PLN_AREA_N == "JURONG WEST")
```
:::


Plotting target areas:


::: {.cell}

```{.r .cell-code}
par(mfrow=c(2,2))
plot(pg, main = "Ponggol")
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-16-1.png){width=672}
:::
:::


#### 5.5.4.2 Creating ***owin*** object

Now, we will convert these sf objects into owin objects that is required by **spatstat**.


::: {.cell}

```{.r .cell-code}
pg_owin = as.owin(pg) 
tm_owin = as.owin(tm) 
ck_owin = as.owin(ck) 
jw_owin = as.owin(jw)
```
:::


#### 5.5.4.3 Combining childcare points and the study area

By using the code chunk below, we are able to extract childcare that is within the specific region to do our analysis later on.


::: {.cell}

```{.r .cell-code}
childcare_pg_ppp = childcare_ppp_jit[pg_owin]
childcare_tm_ppp = childcare_ppp_jit[tm_owin]
childcare_ck_ppp = childcare_ppp_jit[ck_owin]
childcare_jw_ppp = childcare_ppp_jit[jw_owin]
```
:::


rescale.ppp()


::: {.cell}

```{.r .cell-code}
childcare_pg_ppp.km = rescale.ppp(childcare_pg_ppp, 1000, "km") 
childcare_tm_ppp.km = rescale.ppp(childcare_tm_ppp, 1000, "km") 
childcare_ck_ppp.km = rescale.ppp(childcare_ck_ppp, 1000, "km") 
childcare_jw_ppp.km = rescale.ppp(childcare_jw_ppp, 1000, "km")
```
:::


Plot


::: {.cell}

```{.r .cell-code}
par(mfrow=c(2,2)) 
plot(childcare_pg_ppp.km, main="Punggol") 
plot(childcare_tm_ppp.km, main="Tampines") 
plot(childcare_ck_ppp.km, main="Choa Chu Kang") 
plot(childcare_jw_ppp.km, main="Jurong West")
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-20-1.png){width=672}
:::
:::


## **5.6**


::: {.cell}

:::


## **Second-order Spatial Point Patterns Analysis**

## **5.7 Analysing Spatial Point Process Using G-Function**

The G function measures the distribution of the distances from an arbitrary event to its nearest event. In this section, you will learn how to compute G-function estimation by using [*Gest()*](https://rdrr.io/cran/spatstat/man/Gest.html) of **spatstat** package. You will also learn how to perform monta carlo simulation test using [*envelope()*](https://rdrr.io/cran/spatstat/man/envelope.html) of **spatstat** package.

### **5.7.1 Choa Chu Kang planning area**

#### 5.7.1.1 Computing G-function estimation

The code chunk below is used to compute G-function using *Gest()* of **spatat** package.


::: {.cell}

```{.r .cell-code}
G_CK = Gest(childcare_ck_ppp, correction = "border")
plot(G_CK, xlim=c(0,500))
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-22-1.png){width=672}
:::
:::


#### 5.7.1.2 Performing Complete Spatial Randomness Test

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Choa Chu Kang are randomly distributed.

H1= The distribution of childcare services at Choa Chu Kang are not randomly distributed.

The null hypothesis will be rejected if p-value is smaller than alpha value of 0.001.

Monte Carlo test with G-function


::: {.cell}

```{.r .cell-code}
G_CK.csr <- envelope(childcare_ck_ppp, Gest, nsim = 999)
```

::: {.cell-output .cell-output-stdout}

```
Generating 999 simulations of CSR  ...
1, 2, 3, ......10.........20.........30.........40.........50.........60..
.......70.........80.........90.........100.........110.........120.........130
.........140.........150.........160.........170.........180.........190........
.200.........210.........220.........230.........240.........250.........260......
...270.........280.........290.........300.........310.........320.........330....
.....340.........350.........360.........370.........380.........390.........400..
.......410.........420.........430.........440.........450.........460.........470
.........480.........490.........500.........510.........520.........530........
.540.........550.........560.........570.........580.........590.........600......
...610.........620.........630.........640.........650.........660.........670....
.....680.........690.........700.........710.........720.........730.........740..
.......750.........760.........770.........780.........790.........800.........810
.........820.........830.........840.........850.........860.........870........
.880.........890.........900.........910.........920.........930.........940......
...950.........960.........970.........980.........990........
999.

Done.
```


:::
:::

::: {.cell}

```{.r .cell-code}
plot(G_CK.csr)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-24-1.png){width=672}
:::
:::


### **5.7.2 Tampines planning area**

#### 5.7.2.1 Computing G-function estimation


::: {.cell}

```{.r .cell-code}
G_tm = Gest(childcare_tm_ppp, correction = "best")
plot(G_tm)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-25-1.png){width=672}
:::
:::


#### 5.7.2.2 Performing Complete Spatial Randomness Test

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Tampines are randomly distributed.

H1= The distribution of childcare services at Tampines are not randomly distributed.

The null hypothesis will be rejected is p-value is smaller than alpha value of 0.001.

The code chunk below is used to perform the hypothesis testing.


::: {.cell}

```{.r .cell-code}
G_tm.csr <- envelope(childcare_tm_ppp, Gest, correction = "all", nsim = 999)
```

::: {.cell-output .cell-output-stdout}

```
Generating 999 simulations of CSR  ...
1, 2, 3, ......10.........20.........30.........40.........50.........60..
.......70.........80.........90.........100.........110.........120.........130
.........140.........150.........160.........170.........180.........190........
.200.........210.........220.........230.........240.........250.........260......
...270.........280.........290.........300.........310.........320.........330....
.....340.........350.........360.........370.........380.........390.........400..
.......410.........420.........430.........440.........450.........460.........470
.........480.........490.........500.........510.........520.........530........
.540.........550.........560.........570.........580.........590.........600......
...610.........620.........630.........640.........650.........660.........670....
.....680.........690.........700.........710.........720.........730.........740..
.......750.........760.........770.........780.........790.........800.........810
.........820.........830.........840.........850.........860.........870........
.880.........890.........900.........910.........920.........930.........940......
...950.........960.........970.........980.........990........
999.

Done.
```


:::
:::

::: {.cell}

```{.r .cell-code}
plot(G_tm.csr)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-27-1.png){width=672}
:::
:::


## **5.8 Analysing Spatial Point Process Using F-Function**

The F function estimates the empty space function F(r) or its hazard rate h(r) from a point pattern in a window of arbitrary shape. In this section, you will learn how to compute F-function estimation by using [*Fest()*](https://rdrr.io/cran/spatstat/man/Fest.html) of **spatstat** package. You will also learn how to perform monta carlo simulation test using [*envelope()*](https://rdrr.io/cran/spatstat/man/envelope.html) of **spatstat** package.

### **5.8.1 Choa Chu Kang planning area**

#### 5.8.1.1 Computing F-function estimation

The code chunk below is used to compute F-function using *Fest()* of **spatat** package.


::: {.cell}

```{.r .cell-code}
F_CK = Fest(childcare_ck_ppp)
plot(F_CK)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-28-1.png){width=672}
:::
:::


### **5.8.2 Performing Complete Spatial Randomness Test**

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Choa Chu Kang are randomly distributed.

H1= The distribution of childcare services at Choa Chu Kang are not randomly distributed.

The null hypothesis will be rejected if p-value is smaller than alpha value of 0.001.

Monte Carlo test with F-function


::: {.cell}

```{.r .cell-code}
F_CK.csr <- envelope(childcare_ck_ppp, Fest, nsim = 999)
```

::: {.cell-output .cell-output-stdout}

```
Generating 999 simulations of CSR  ...
1, 2, 3, ......10.........20.........30.........40.........50.........60..
.......70.........80.........90.........100.........110.........120.........130
.........140.........150.........160.........170.........180.........190........
.200.........210.........220.........230.........240.........250.........260......
...270.........280.........290.........300.........310.........320.........330....
.....340.........350.........360.........370.........380.........390.........400..
.......410.........420.........430.........440.........450.........460.........470
.........480.........490.........500.........510.........520.........530........
.540.........550.........560.........570.........580.........590.........600......
...610.........620.........630.........640.........650.........660.........670....
.....680.........690.........700.........710.........720.........730.........740..
.......750.........760.........770.........780.........790.........800.........810
.........820.........830.........840.........850.........860.........870........
.880.........890.........900.........910.........920.........930.........940......
...950.........960.........970.........980.........990........
999.

Done.
```


:::
:::

::: {.cell}

```{.r .cell-code}
plot(F_CK.csr)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-30-1.png){width=672}
:::
:::


### **5.8.3 Tampines planning area**

#### 5.8.3.1 Computing F-function estimation

Monte Carlo test with F-function


::: {.cell}

```{.r .cell-code}
F_tm = Fest(childcare_tm_ppp, correction = "best")
plot(F_tm)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-31-1.png){width=672}
:::
:::


#### 5.8.3.2 Performing Complete Spatial Randomness Test

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Tampines are randomly distributed.

H1= The distribution of childcare services at Tampines are not randomly distributed.

The null hypothesis will be rejected is p-value is smaller than alpha value of 0.001.

The code chunk below is used to perform the hypothesis testing.


::: {.cell}

```{.r .cell-code}
F_tm.csr <- envelope(childcare_tm_ppp, Fest, correction = "all", nsim = 999)
```

::: {.cell-output .cell-output-stdout}

```
Generating 999 simulations of CSR  ...
1, 2, 3, ......10.........20.........30.........40.........50.........60..
.......70.........80.........90.........100.........110.........120.........130
.........140.........150.........160.........170.........180.........190........
.200.........210.........220.........230.........240.........250.........260......
...270.........280.........290.........300.........310.........320.........330....
.....340.........350.........360.........370.........380.........390.........400..
.......410.........420.........430.........440.........450.........460.........470
.........480.........490.........500.........510.........520.........530........
.540.........550.........560.........570.........580.........590.........600......
...610.........620.........630.........640.........650.........660.........670....
.....680.........690.........700.........710.........720.........730.........740..
.......750.........760.........770.........780.........790.........800.........810
.........820.........830.........840.........850.........860.........870........
.880.........890.........900.........910.........920.........930.........940......
...950.........960.........970.........980.........990........
999.

Done.
```


:::
:::

::: {.cell}

```{.r .cell-code}
plot(F_tm.csr)
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-33-1.png){width=672}
:::
:::


## **5.9 Analysing Spatial Point Process Using K-Function**

K-function measures the number of events found up to a given distance of any particular event. In this section, you will learn how to compute K-function estimates by using [*Kest()*](https://rdrr.io/cran/spatstat/man/Kest.html) of **spatstat** package. You will also learn how to perform monta carlo simulation test using *envelope()* of spatstat package.

### **5.9.1 Choa Chu Kang planning area**

#### 5.9.1.1 Computing K-fucntion estimate


::: {.cell}

```{.r .cell-code}
K_ck = Kest(childcare_ck_ppp, correction = "Ripley")
plot(K_ck, . -r ~ r, ylab= "K(d)-r", xlab = "d(m)")
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-34-1.png){width=672}
:::
:::


#### 5.9.1.2 Performing Complete Spatial Randomness Test

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Choa Chu Kang are randomly distributed.

H1= The distribution of childcare services at Choa Chu Kang are not randomly distributed.

The null hypothesis will be rejected if p-value is smaller than alpha value of 0.001.

The code chunk below is used to perform the hypothesis testing.


::: {.cell}

```{.r .cell-code}
K_ck.csr <- envelope(childcare_ck_ppp, Kest, nsim = 99, rank = 1, glocal=TRUE)
```

::: {.cell-output .cell-output-stdout}

```
Generating 99 simulations of CSR  ...
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80,
81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 
99.

Done.
```


:::
:::

::: {.cell}

```{.r .cell-code}
plot(K_ck.csr, . - r ~ r, xlab="d", ylab="K(d)-r")
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-36-1.png){width=672}
:::
:::


### **5.9.2 Tampines planning area**

#### 5.9.2.1 Computing K-function estimation


::: {.cell}

```{.r .cell-code}
K_tm = Kest(childcare_tm_ppp, correction = "Ripley")
plot(K_tm, . -r ~ r, 
     ylab= "K(d)-r", xlab = "d(m)", 
     xlim=c(0,1000))
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-37-1.png){width=672}
:::
:::


5.9.2.2 Performing Complete Spatial Randomness Test

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Tampines are randomly distributed.

H1= The distribution of childcare services at Tampines are not randomly distributed.

The null hypothesis will be rejected if p-value is smaller than alpha value of 0.001.

The code chunk below is used to perform the hypothesis testing.


::: {.cell}

```{.r .cell-code}
K_tm.csr <- envelope(childcare_tm_ppp, Kest, nsim = 99, rank = 1, glocal=TRUE)
```

::: {.cell-output .cell-output-stdout}

```
Generating 99 simulations of CSR  ...
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80,
81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 
99.

Done.
```


:::
:::

::: {.cell}

```{.r .cell-code}
plot(K_tm.csr, . - r ~ r, 
     xlab="d", ylab="K(d)-r", xlim=c(0,500))
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-39-1.png){width=672}
:::
:::


## **5.10 Analysing Spatial Point Process Using L-Function**

In this section, you will learn how to compute L-function estimation by using [*Lest()*](https://rdrr.io/cran/spatstat/man/Lest.html) of **spatstat** package. You will also learn how to perform monta carlo simulation test using *envelope()* of spatstat package.

### **5.10.1 Choa Chu Kang planning area**

#### 5.10.1.1 Computing L Function estimation


::: {.cell}

```{.r .cell-code}
L_ck = Lest(childcare_ck_ppp, correction = "Ripley")
plot(L_ck, . -r ~ r, 
     ylab= "L(d)-r", xlab = "d(m)")
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-40-1.png){width=672}
:::
:::


#### 5.10.1.2 Performing Complete Spatial Randomness Test

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Choa Chu Kang are randomly distributed.

H1= The distribution of childcare services at Choa Chu Kang are not randomly distributed.

The null hypothesis will be rejected if p-value if smaller than alpha value of 0.001.

The code chunk below is used to perform the hypothesis testing.


::: {.cell}

```{.r .cell-code}
L_ck.csr <- envelope(childcare_ck_ppp, Lest, nsim = 99, rank = 1, glocal=TRUE)
```

::: {.cell-output .cell-output-stdout}

```
Generating 99 simulations of CSR  ...
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80,
81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 
99.

Done.
```


:::
:::

::: {.cell}

```{.r .cell-code}
L_ck.csr <- envelope(childcare_ck_ppp, Lest, nsim = 99, rank = 1, glocal=TRUE)
```

::: {.cell-output .cell-output-stdout}

```
Generating 99 simulations of CSR  ...
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80,
81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 
99.

Done.
```


:::
:::


### **5.10.2 Tampines planning area**

#### 5.10.2.1 Computing L-function estimate


::: {.cell}

```{.r .cell-code}
L_tm = Lest(childcare_tm_ppp, correction = "Ripley")
plot(L_tm, . -r ~ r, 
     ylab= "L(d)-r", xlab = "d(m)", 
     xlim=c(0,1000))
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-43-1.png){width=672}
:::
:::


#### 5.10.2.2 Performing Complete Spatial Randomness Test

To confirm the observed spatial patterns above, a hypothesis test will be conducted. The hypothesis and test are as follows:

Ho = The distribution of childcare services at Tampines are randomly distributed.

H1= The distribution of childcare services at Tampines are not randomly distributed.

The null hypothesis will be rejected if p-value is smaller than alpha value of 0.001.

The code chunk below will be used to perform the hypothesis testing.


::: {.cell}

```{.r .cell-code}
L_tm.csr <- envelope(childcare_tm_ppp, Lest, nsim = 99, rank = 1, glocal=TRUE)
```

::: {.cell-output .cell-output-stdout}

```
Generating 99 simulations of CSR  ...
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80,
81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 
99.

Done.
```


:::
:::


Plot:


::: {.cell}

```{.r .cell-code}
plot(L_tm.csr, . - r ~ r, 
     xlab="d", ylab="L(d)-r", xlim=c(0,500))
```

::: {.cell-output-display}
![](Hands-on_Ex02b_files/figure-html/unnamed-chunk-45-1.png){width=672}
:::
:::


## 6.0 Reference

Prof T.S. Kam - [Chapter 5](https://r4gdsa.netlify.app/chap05.html): R for Geospatial Data Science and Analytics
