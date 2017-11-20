xkcd plot of german weather
================

``` r
#devtools::install_github('hadley/ggplot2')
library(plotly)
library(leaflet)
library(htmltools)
library(tidyverse)
library(xkcd)
library(extrafont)
library(ggrepel)
```

So [Maële Salmon](http://www.masalmon.eu/2017/11/16/wheretoliveus/) started this trend how to determine your city of choice based on your climate preference, inspired by [this xkcd graph](https://xkcd.com/1916/). We already know that the [Spanish](https://twitter.com/claudiaguirao/status/931615734521909248) islands are our all-time favorite (you could have asked the German tourists) that you really should only go to [Iceland](https://twitter.com/matamix/status/932192147062784000) because of the landscape and that the weather is the same in the [Netherlands](https://twitter.com/RMHoge/status/932526164668829696), no matter where you go.

Different to the above mentioned analysis, we just take a look at the temperature to determine the ideal german city. The reason for this is the way the German Meteorological Service provides data. You will certainly get the necessary data to calculate the [humidex](https://en.wikipedia.org/wiki/Humidex), but the time I want to put into this small project is limited. For this reason, I reduce the analysis to the data set of average temperatures, downloaded from the [open data platform of the dwd](ftp://ftp-cdc.dwd.de/pub/CDC/regional_averages_DE/).

If you just forgot where Germany is located - here is a little help:

``` r
# Set coordinates (http://latitude.to/map/de/germany)
lng = 10.4541194
lat = 51.1642292

leaflet() %>% addTiles() %>%
  setView(lng=lng, lat=lat, zoom=4) %>%
  addProviderTiles(providers$Stamen.Toner) %>%
  addMarkers(lng=lng, lat=lat, 
             label = paste0("Coordinates: ",as.character(lat)," ",as.character(lng)))
```

<!--html_preserve-->

<script type="application/json" data-for="htmlwidget-960e5e28059d73a2319d">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"maxNativeZoom":null,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"continuousWorld":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addProviderTiles","args":["Stamen.Toner",null,null,{"errorTileUrl":"","noWrap":false,"zIndex":null,"unloadInvisibleTiles":null,"updateWhenIdle":null,"detectRetina":false,"reuseTiles":false}]},{"method":"addMarkers","args":[51.1642292,10.4541194,null,null,null,{"clickable":true,"draggable":false,"keyboard":true,"title":"","alt":"","zIndexOffset":0,"opacity":1,"riseOnHover":false,"riseOffset":250},null,null,null,null,"Coordinates: 51.1642292 10.4541194",null,null]}],"setView":[[51.1642292,10.4541194],4,[]],"limits":{"lat":[51.1642292,51.1642292],"lng":[10.4541194,10.4541194]}},"evals":[],"jsHooks":[]}</script>
<!--/html_preserve-->
``` r
#saveWidget(m, file="m.html")
```

Import the Data
---------------

We have the seasonal average temperature as well as precipitation of the german federal states, derived from the gridded fields covering Germany for the period 1881 - 2017.

``` r
rm(list=ls())
air_temp <- read.csv("data/air_temperature_mean.txt", sep=";")
air_temp %>% gather(key = "location", value = "air_temp", -Jahr, -season) -> air_temp

rain <- read.csv("data/rain.txt", sep=";")
rain %>% gather(key = "location", value = "rain", -Jahr, -season) -> rain
```

plot like xkcd
--------------

I had to manually download the [xkcd font](http://simonsoftware.se/other/xkcd.ttf) and copy it to the font app on my Mac.

We use the fact that we have data from 1881 to the present day and compare 2016 with 1916:

``` r
seas <- c("summer", "winter")
delete <- c("Deutschland", "X")
air_temp %>% 
  filter(season %in% seas) %>%
  filter(!location %in% delete) %>%
  #filter(Jahr == 2016) %>%
  spread(season, air_temp) %>%
  filter(!is.na(winter)) %>%
  filter(!is.na(summer)) -> plot.df1
```

``` r
plot.df1 %>%
  filter(Jahr == 1916 | Jahr == 2016) %>%
  ggplot(aes(summer, winter, color=as.factor(Jahr))) +
  geom_point() +
  geom_text_repel(aes(label = location), family = "xkcd", 
                   max.iter = 50000, size = 3) +
  
  ggtitle("Where to live in Germany - 1916 vs. 2016",
          subtitle = "Data from DWD") +
  labs(color = "Year", 
       x= "Summer mean temperature in Celsius degrees", 
       y = "Winter mean temperature in Celsius degrees") +
  theme_xkcd() +
  theme(text = element_text(size = 13, family = "xkcd"))
```

![](README_files/figure-markdown_github-ascii_identifiers/scatterplot-1.png)

Let's see if this average temperature rise can be tracked over time.

``` r
seas <- c("summer", "winter")
delete <- c("Deutschland")

air_temp %>% 
  filter(season %in% seas) %>%
  filter(location %in% delete) -> plot.df2
```

``` r
g <- plot.df2 %>%
  ggplot(aes(Jahr, air_temp, color=season)) +
  geom_line() +
  ggtitle("Average temperature in Germany - 1881-2017",
          subtitle = "Data from DWD") +
  labs(
       x= "", 
       y = "Average temperature in Celsius degrees") +
  theme_xkcd() +
  theme(text = element_text(size = 13, family = "xkcd"))

ggplotly(g, tooltip = c("Jahr","air_temp"))
```

<!--html_preserve-->

<script type="application/json" data-for="33935d3aaa3d">{"x":{"data":[{"x":[1881,1882,1883,1884,1885,1886,1887,1888,1889,1890,1891,1892,1893,1894,1895,1896,1897,1898,1899,1900,1901,1902,1903,1904,1905,1906,1907,1908,1909,1910,1911,1912,1913,1914,1915,1916,1917,1918,1919,1920,1921,1922,1923,1924,1925,1926,1927,1928,1929,1930,1931,1932,1933,1934,1935,1936,1937,1938,1939,1940,1941,1942,1943,1944,1945,1946,1947,1948,1949,1950,1951,1952,1953,1954,1955,1956,1957,1958,1959,1960,1961,1962,1963,1964,1965,1966,1967,1968,1969,1970,1971,1972,1973,1974,1975,1976,1977,1978,1979,1980,1981,1982,1983,1984,1985,1986,1987,1988,1989,1990,1991,1992,1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017],"y":[16.5,15.3,16.3,16.1,16.2,16,16.6,15.2,16.8,15.3,15.3,16.4,16.7,15.8,16.5,16,16.9,15.7,16.3,16.8,16.8,15.3,15.5,16.8,17.4,16.1,14.9,16.2,15.1,16,17.8,15.8,14.7,16.4,16.3,14.9,17.6,15.1,15.1,15.8,16.8,15.6,15.2,15.5,16.4,15.7,15.9,16.2,16.4,16.8,16.1,17.1,16.4,17,17.2,16.4,16.8,16.8,16.9,15.8,16.6,16,16.7,17.1,16.9,16.4,18.5,16.1,16.2,17.7,16.5,17.1,16.6,15.4,16.2,14.7,16.5,16.1,17.6,15.8,15.6,15,16.5,16.9,15.1,16,16.6,16.2,16.6,16.7,16.6,16,17,15.5,17.3,17.6,16,15.1,15.8,15.5,16,17.5,18.3,15.5,15.7,16.4,15.4,16.3,16.7,16.7,16.9,18.3,15.8,18.4,17.6,16.2,17.6,16.5,17.1,16.6,17.2,18,19.7,16.8,16.7,18.1,17.2,17.4,17.2,17.8,16.8,17.1,17.7,17.1,18.4,17.8,17.9],"text":["Jahr: 1881<br />air_temp: 16.5","Jahr: 1882<br />air_temp: 15.3","Jahr: 1883<br />air_temp: 16.3","Jahr: 1884<br />air_temp: 16.1","Jahr: 1885<br />air_temp: 16.2","Jahr: 1886<br />air_temp: 16.0","Jahr: 1887<br />air_temp: 16.6","Jahr: 1888<br />air_temp: 15.2","Jahr: 1889<br />air_temp: 16.8","Jahr: 1890<br />air_temp: 15.3","Jahr: 1891<br />air_temp: 15.3","Jahr: 1892<br />air_temp: 16.4","Jahr: 1893<br />air_temp: 16.7","Jahr: 1894<br />air_temp: 15.8","Jahr: 1895<br />air_temp: 16.5","Jahr: 1896<br />air_temp: 16.0","Jahr: 1897<br />air_temp: 16.9","Jahr: 1898<br />air_temp: 15.7","Jahr: 1899<br />air_temp: 16.3","Jahr: 1900<br />air_temp: 16.8","Jahr: 1901<br />air_temp: 16.8","Jahr: 1902<br />air_temp: 15.3","Jahr: 1903<br />air_temp: 15.5","Jahr: 1904<br />air_temp: 16.8","Jahr: 1905<br />air_temp: 17.4","Jahr: 1906<br />air_temp: 16.1","Jahr: 1907<br />air_temp: 14.9","Jahr: 1908<br />air_temp: 16.2","Jahr: 1909<br />air_temp: 15.1","Jahr: 1910<br />air_temp: 16.0","Jahr: 1911<br />air_temp: 17.8","Jahr: 1912<br />air_temp: 15.8","Jahr: 1913<br />air_temp: 14.7","Jahr: 1914<br />air_temp: 16.4","Jahr: 1915<br />air_temp: 16.3","Jahr: 1916<br />air_temp: 14.9","Jahr: 1917<br />air_temp: 17.6","Jahr: 1918<br />air_temp: 15.1","Jahr: 1919<br />air_temp: 15.1","Jahr: 1920<br />air_temp: 15.8","Jahr: 1921<br />air_temp: 16.8","Jahr: 1922<br />air_temp: 15.6","Jahr: 1923<br />air_temp: 15.2","Jahr: 1924<br />air_temp: 15.5","Jahr: 1925<br />air_temp: 16.4","Jahr: 1926<br />air_temp: 15.7","Jahr: 1927<br />air_temp: 15.9","Jahr: 1928<br />air_temp: 16.2","Jahr: 1929<br />air_temp: 16.4","Jahr: 1930<br />air_temp: 16.8","Jahr: 1931<br />air_temp: 16.1","Jahr: 1932<br />air_temp: 17.1","Jahr: 1933<br />air_temp: 16.4","Jahr: 1934<br />air_temp: 17.0","Jahr: 1935<br />air_temp: 17.2","Jahr: 1936<br />air_temp: 16.4","Jahr: 1937<br />air_temp: 16.8","Jahr: 1938<br />air_temp: 16.8","Jahr: 1939<br />air_temp: 16.9","Jahr: 1940<br />air_temp: 15.8","Jahr: 1941<br />air_temp: 16.6","Jahr: 1942<br />air_temp: 16.0","Jahr: 1943<br />air_temp: 16.7","Jahr: 1944<br />air_temp: 17.1","Jahr: 1945<br />air_temp: 16.9","Jahr: 1946<br />air_temp: 16.4","Jahr: 1947<br />air_temp: 18.5","Jahr: 1948<br />air_temp: 16.1","Jahr: 1949<br />air_temp: 16.2","Jahr: 1950<br />air_temp: 17.7","Jahr: 1951<br />air_temp: 16.5","Jahr: 1952<br />air_temp: 17.1","Jahr: 1953<br />air_temp: 16.6","Jahr: 1954<br />air_temp: 15.4","Jahr: 1955<br />air_temp: 16.2","Jahr: 1956<br />air_temp: 14.7","Jahr: 1957<br />air_temp: 16.5","Jahr: 1958<br />air_temp: 16.1","Jahr: 1959<br />air_temp: 17.6","Jahr: 1960<br />air_temp: 15.8","Jahr: 1961<br />air_temp: 15.6","Jahr: 1962<br />air_temp: 15.0","Jahr: 1963<br />air_temp: 16.5","Jahr: 1964<br />air_temp: 16.9","Jahr: 1965<br />air_temp: 15.1","Jahr: 1966<br />air_temp: 16.0","Jahr: 1967<br />air_temp: 16.6","Jahr: 1968<br />air_temp: 16.2","Jahr: 1969<br />air_temp: 16.6","Jahr: 1970<br />air_temp: 16.7","Jahr: 1971<br />air_temp: 16.6","Jahr: 1972<br />air_temp: 16.0","Jahr: 1973<br />air_temp: 17.0","Jahr: 1974<br />air_temp: 15.5","Jahr: 1975<br />air_temp: 17.3","Jahr: 1976<br />air_temp: 17.6","Jahr: 1977<br />air_temp: 16.0","Jahr: 1978<br />air_temp: 15.1","Jahr: 1979<br />air_temp: 15.8","Jahr: 1980<br />air_temp: 15.5","Jahr: 1981<br />air_temp: 16.0","Jahr: 1982<br />air_temp: 17.5","Jahr: 1983<br />air_temp: 18.3","Jahr: 1984<br />air_temp: 15.5","Jahr: 1985<br />air_temp: 15.7","Jahr: 1986<br />air_temp: 16.4","Jahr: 1987<br />air_temp: 15.4","Jahr: 1988<br />air_temp: 16.3","Jahr: 1989<br />air_temp: 16.7","Jahr: 1990<br />air_temp: 16.7","Jahr: 1991<br />air_temp: 16.9","Jahr: 1992<br />air_temp: 18.3","Jahr: 1993<br />air_temp: 15.8","Jahr: 1994<br />air_temp: 18.4","Jahr: 1995<br />air_temp: 17.6","Jahr: 1996<br />air_temp: 16.2","Jahr: 1997<br />air_temp: 17.6","Jahr: 1998<br />air_temp: 16.5","Jahr: 1999<br />air_temp: 17.1","Jahr: 2000<br />air_temp: 16.6","Jahr: 2001<br />air_temp: 17.2","Jahr: 2002<br />air_temp: 18.0","Jahr: 2003<br />air_temp: 19.7","Jahr: 2004<br />air_temp: 16.8","Jahr: 2005<br />air_temp: 16.7","Jahr: 2006<br />air_temp: 18.1","Jahr: 2007<br />air_temp: 17.2","Jahr: 2008<br />air_temp: 17.4","Jahr: 2009<br />air_temp: 17.2","Jahr: 2010<br />air_temp: 17.8","Jahr: 2011<br />air_temp: 16.8","Jahr: 2012<br />air_temp: 17.1","Jahr: 2013<br />air_temp: 17.7","Jahr: 2014<br />air_temp: 17.1","Jahr: 2015<br />air_temp: 18.4","Jahr: 2016<br />air_temp: 17.8","Jahr: 2017<br />air_temp: 17.9"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(248,118,109,1)","dash":"solid"},"hoveron":"points","name":"summer","legendgroup":"summer","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1882,1883,1884,1885,1886,1887,1888,1889,1890,1891,1892,1893,1894,1895,1896,1897,1898,1899,1900,1901,1902,1903,1904,1905,1906,1907,1908,1909,1910,1911,1912,1913,1914,1915,1916,1917,1918,1919,1920,1921,1922,1923,1924,1925,1926,1927,1928,1929,1930,1931,1932,1933,1934,1935,1936,1937,1938,1939,1940,1941,1942,1943,1944,1945,1946,1947,1948,1949,1950,1951,1952,1953,1954,1955,1956,1957,1958,1959,1960,1961,1962,1963,1964,1965,1966,1967,1968,1969,1970,1971,1972,1973,1974,1975,1976,1977,1978,1979,1980,1981,1982,1983,1984,1985,1986,1987,1988,1989,1990,1991,1992,1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017],"y":[1,0.9,2.1,0.6,-1.5,-1.3,-1.8,-1.5,-0.7,-3.5,0.2,-2.4,0,-3.4,-0.4,-0.7,1.5,2.4,-0.5,-1.7,0.8,0.7,-0.3,0.6,0.8,-1.3,0.2,-1.5,1.9,0.9,1.3,1.4,0.7,1.6,2.7,-1.5,0.1,1.4,2,2,-1.1,1.6,-2.4,2.5,1.3,1.1,0.5,-4.9,1.9,-0.1,0,-0.6,-0.8,2,1.2,1,0.7,0.8,-5,-2.8,-3.9,1.5,0.7,-0.1,0.6,-4.5,1.7,1.3,1.6,0.4,0.9,-0.5,-1.4,-0.1,-2.3,2,0.7,0.8,0.8,1.6,0.5,-5.5,-1.7,0,1.2,1.8,0,-1.2,-2.8,0.3,1.1,0.6,2.2,3.6,1,0.8,0.5,-2,1.1,-0.4,-1.5,1.5,0.5,-2.5,-0.9,-1.4,2.6,3.1,3.6,-0.1,1.5,1,2,2.8,-2.3,-0.3,3,1.3,2.4,2.1,2,-0.6,1.4,0.7,-0.7,4.4,3,-0.2,-1.3,-0.6,1.1,0.3,3.3,1.9,3.6,1],"text":["Jahr: 1882<br />air_temp:  1.0","Jahr: 1883<br />air_temp:  0.9","Jahr: 1884<br />air_temp:  2.1","Jahr: 1885<br />air_temp:  0.6","Jahr: 1886<br />air_temp: -1.5","Jahr: 1887<br />air_temp: -1.3","Jahr: 1888<br />air_temp: -1.8","Jahr: 1889<br />air_temp: -1.5","Jahr: 1890<br />air_temp: -0.7","Jahr: 1891<br />air_temp: -3.5","Jahr: 1892<br />air_temp:  0.2","Jahr: 1893<br />air_temp: -2.4","Jahr: 1894<br />air_temp:  0.0","Jahr: 1895<br />air_temp: -3.4","Jahr: 1896<br />air_temp: -0.4","Jahr: 1897<br />air_temp: -0.7","Jahr: 1898<br />air_temp:  1.5","Jahr: 1899<br />air_temp:  2.4","Jahr: 1900<br />air_temp: -0.5","Jahr: 1901<br />air_temp: -1.7","Jahr: 1902<br />air_temp:  0.8","Jahr: 1903<br />air_temp:  0.7","Jahr: 1904<br />air_temp: -0.3","Jahr: 1905<br />air_temp:  0.6","Jahr: 1906<br />air_temp:  0.8","Jahr: 1907<br />air_temp: -1.3","Jahr: 1908<br />air_temp:  0.2","Jahr: 1909<br />air_temp: -1.5","Jahr: 1910<br />air_temp:  1.9","Jahr: 1911<br />air_temp:  0.9","Jahr: 1912<br />air_temp:  1.3","Jahr: 1913<br />air_temp:  1.4","Jahr: 1914<br />air_temp:  0.7","Jahr: 1915<br />air_temp:  1.6","Jahr: 1916<br />air_temp:  2.7","Jahr: 1917<br />air_temp: -1.5","Jahr: 1918<br />air_temp:  0.1","Jahr: 1919<br />air_temp:  1.4","Jahr: 1920<br />air_temp:  2.0","Jahr: 1921<br />air_temp:  2.0","Jahr: 1922<br />air_temp: -1.1","Jahr: 1923<br />air_temp:  1.6","Jahr: 1924<br />air_temp: -2.4","Jahr: 1925<br />air_temp:  2.5","Jahr: 1926<br />air_temp:  1.3","Jahr: 1927<br />air_temp:  1.1","Jahr: 1928<br />air_temp:  0.5","Jahr: 1929<br />air_temp: -4.9","Jahr: 1930<br />air_temp:  1.9","Jahr: 1931<br />air_temp: -0.1","Jahr: 1932<br />air_temp:  0.0","Jahr: 1933<br />air_temp: -0.6","Jahr: 1934<br />air_temp: -0.8","Jahr: 1935<br />air_temp:  2.0","Jahr: 1936<br />air_temp:  1.2","Jahr: 1937<br />air_temp:  1.0","Jahr: 1938<br />air_temp:  0.7","Jahr: 1939<br />air_temp:  0.8","Jahr: 1940<br />air_temp: -5.0","Jahr: 1941<br />air_temp: -2.8","Jahr: 1942<br />air_temp: -3.9","Jahr: 1943<br />air_temp:  1.5","Jahr: 1944<br />air_temp:  0.7","Jahr: 1945<br />air_temp: -0.1","Jahr: 1946<br />air_temp:  0.6","Jahr: 1947<br />air_temp: -4.5","Jahr: 1948<br />air_temp:  1.7","Jahr: 1949<br />air_temp:  1.3","Jahr: 1950<br />air_temp:  1.6","Jahr: 1951<br />air_temp:  0.4","Jahr: 1952<br />air_temp:  0.9","Jahr: 1953<br />air_temp: -0.5","Jahr: 1954<br />air_temp: -1.4","Jahr: 1955<br />air_temp: -0.1","Jahr: 1956<br />air_temp: -2.3","Jahr: 1957<br />air_temp:  2.0","Jahr: 1958<br />air_temp:  0.7","Jahr: 1959<br />air_temp:  0.8","Jahr: 1960<br />air_temp:  0.8","Jahr: 1961<br />air_temp:  1.6","Jahr: 1962<br />air_temp:  0.5","Jahr: 1963<br />air_temp: -5.5","Jahr: 1964<br />air_temp: -1.7","Jahr: 1965<br />air_temp:  0.0","Jahr: 1966<br />air_temp:  1.2","Jahr: 1967<br />air_temp:  1.8","Jahr: 1968<br />air_temp:  0.0","Jahr: 1969<br />air_temp: -1.2","Jahr: 1970<br />air_temp: -2.8","Jahr: 1971<br />air_temp:  0.3","Jahr: 1972<br />air_temp:  1.1","Jahr: 1973<br />air_temp:  0.6","Jahr: 1974<br />air_temp:  2.2","Jahr: 1975<br />air_temp:  3.6","Jahr: 1976<br />air_temp:  1.0","Jahr: 1977<br />air_temp:  0.8","Jahr: 1978<br />air_temp:  0.5","Jahr: 1979<br />air_temp: -2.0","Jahr: 1980<br />air_temp:  1.1","Jahr: 1981<br />air_temp: -0.4","Jahr: 1982<br />air_temp: -1.5","Jahr: 1983<br />air_temp:  1.5","Jahr: 1984<br />air_temp:  0.5","Jahr: 1985<br />air_temp: -2.5","Jahr: 1986<br />air_temp: -0.9","Jahr: 1987<br />air_temp: -1.4","Jahr: 1988<br />air_temp:  2.6","Jahr: 1989<br />air_temp:  3.1","Jahr: 1990<br />air_temp:  3.6","Jahr: 1991<br />air_temp: -0.1","Jahr: 1992<br />air_temp:  1.5","Jahr: 1993<br />air_temp:  1.0","Jahr: 1994<br />air_temp:  2.0","Jahr: 1995<br />air_temp:  2.8","Jahr: 1996<br />air_temp: -2.3","Jahr: 1997<br />air_temp: -0.3","Jahr: 1998<br />air_temp:  3.0","Jahr: 1999<br />air_temp:  1.3","Jahr: 2000<br />air_temp:  2.4","Jahr: 2001<br />air_temp:  2.1","Jahr: 2002<br />air_temp:  2.0","Jahr: 2003<br />air_temp: -0.6","Jahr: 2004<br />air_temp:  1.4","Jahr: 2005<br />air_temp:  0.7","Jahr: 2006<br />air_temp: -0.7","Jahr: 2007<br />air_temp:  4.4","Jahr: 2008<br />air_temp:  3.0","Jahr: 2009<br />air_temp: -0.2","Jahr: 2010<br />air_temp: -1.3","Jahr: 2011<br />air_temp: -0.6","Jahr: 2012<br />air_temp:  1.1","Jahr: 2013<br />air_temp:  0.3","Jahr: 2014<br />air_temp:  3.3","Jahr: 2015<br />air_temp:  1.9","Jahr: 2016<br />air_temp:  3.6","Jahr: 2017<br />air_temp:  1.0"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(0,191,196,1)","dash":"solid"},"hoveron":"points","name":"winter","legendgroup":"winter","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":46.950601909506,"r":7.30593607305936,"b":27.6961394769614,"l":42.0423412204234},"paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"xkcd","size":17.2685761726858},"title":"Average temperature in Germany - 1881-2017","titlefont":{"color":"rgba(0,0,0,1)","family":"xkcd","size":20.7222914072229},"xaxis":{"domain":[0,1],"type":"linear","autorange":false,"range":[1874.2,2023.8],"tickmode":"array","ticktext":["1880","1920","1960","2000"],"tickvals":[1880,1920,1960,2000],"categoryorder":"array","categoryarray":["1880","1920","1960","2000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(0,0,0,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"xkcd","size":13.8148609381486},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":false,"gridcolor":null,"gridwidth":0,"zeroline":false,"anchor":"y","title":"","titlefont":{"color":"rgba(0,0,0,1)","family":"xkcd","size":17.2685761726858},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"type":"linear","autorange":false,"range":[-6.76,20.96],"tickmode":"array","ticktext":["0","10","20"],"tickvals":[0,10,20],"categoryorder":"array","categoryarray":["0","10","20"],"nticks":null,"ticks":"outside","tickcolor":"rgba(0,0,0,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"xkcd","size":13.8148609381486},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":false,"gridcolor":null,"gridwidth":0,"zeroline":false,"anchor":"x","title":"Average temperature in Celsius degrees","titlefont":{"color":"rgba(0,0,0,1)","family":"xkcd","size":17.2685761726858},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"xkcd","size":13.8148609381486},"y":0.897637795275591},"annotations":[{"text":"season","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"xkcd","size":17.2685761726858},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"cloud":false},"source":"A","attrs":{"33937976a8a7":{"x":{},"y":{},"colour":{},"type":"scatter"}},"cur_data":"33937976a8a7","visdat":{"33937976a8a7":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1}},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":{"render":[{"code":"function(el, x) { var ctConfig = crosstalk.var('plotlyCrosstalkOpts').set({\"on\":\"plotly_click\",\"persistent\":false,\"dynamic\":false,\"selectize\":false,\"opacityDim\":0.2,\"selected\":{\"opacity\":1}}); }","data":null}]}}</script>
<!--/html_preserve-->
