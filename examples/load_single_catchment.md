Visualización datos CAMELS-CL descargados desde el explorador
================

Este notebook describe un ejemplo de cómo visualizar los datos de una cuenca de la base de datos CAMELS-CL, utilizando R. Los datos deben descargarse previamente desde el explorador <http://camels.cr2.cl>, para la cuenca en estudio

En caso de que se requiera trabajar con todas las cuencas de la base de datos, se recomienda descargar los datos directamente del repositorio <https://doi.pangaea.de/10.1594/PANGAEA.894885>

### 1) Definir directorios, cuencas y variables a visualizar

``` r
# directorio que contiene los datos descargados desde el explorador
path_folder  <- "~/Downloads"

# cuenca descargada 
gauge_id    <- 5710001 

# series hidro-meteorológicas que se quieren visualizar, e.g., q_mm_day, precip_cr2met_day, tmean_cr2met_day, pet_hargreaves_day, q_mm_mon, precip_cr2met_mon, tmean_cr2met_mon, pet_hargreaves_mon, etc.
var_ts1  <- "q_mm_mon"
var_ts2  <- "precip_cr2met_mon"

# fechas en que se quiere visualizar las series de tiempo
date_ini <- "1979-01-01"
date_end <- "2016-12-01"
```

### 2) Cargar datos

``` r
# cargar librerías
library(readr)
library(raster)
library(data.table)
library(gridExtra)
library(grid)

# cargar datos cuenca
attrib_csv  <- read_csv(paste(path_folder,"/camels_cl_",gauge_id,"/catchment_attributes.csv",sep=""),col_names = FALSE)
attrib      <- t(as.matrix(attrib_csv[,2]))
colnames(attrib) <- t(as.matrix(attrib_csv[,1]))
attrib <- data.frame(attrib,stringsAsFactors = FALSE)

ts1           <- read_csv(paste(path_folder,"/camels_cl_",gauge_id,"/", var_ts1, ".csv",sep=""))
colnames(ts1) <- c("date",var_ts1)
ts2    <- read_csv(paste(path_folder,"/camels_cl_",gauge_id,"/", var_ts2, ".csv",sep=""))
colnames(ts2) <- c("date",var_ts2)
basin  <- shapefile(paste(path_folder,"/camels_cl_",gauge_id,"/polygon/polygon.shp",sep=""))
```

### 3) Generar mapa y resumen de atributos principales

``` r
  # dev.new()
  par(mfrow=c(1,4))
  # plot basin
  plot(basin)
  points(attrib$gauge_lon,attrib$gauge_lat,bg="red",pch=21,cex=2,lwd=2)
  mtext(text = paste("Cuenca ", attrib$gauge_name, " (id ", attrib$gauge_id, ")",sep=""),side = 3, cex=1,font=2,col="gray14",line = 0,adj=0)

  # define and plot catchment attributes (Se pueden elegir diferentes atributos).     
  attrib_table  <- data.frame(round(as.numeric(attrib$area,1)),
                              round(as.numeric(attrib$elev_gauge,0)),
                              round(as.numeric(attrib$elev_mean,0)),
                              round(as.numeric(attrib$elev_max,0)),
                              round(as.numeric(attrib$slope_mean,1)),
                              round(as.numeric(attrib$interv_degree,3)))

  attrib_table <- rbind(attrib_table,
                      c("km2","m s.n.m.","m s.n.m.","m s.n.m.","m/km","-"))
    
  colnames(attrib_table)<- c("Area cuenca",
                             "Elev. punto salida",
                             "Elev. media cuenca",
                             "Elev. máx. cuenca",
                             "Pendiente media",
                             "Grado intervención\nantrópica superficial")
  natt=9
  t1 <- ttheme_default(
      core=list(fg_params = list(fontface=c(rep("plain", natt),rep("plain", natt)),
                                 fontfamily="Helvetica",x=c(rep(.5,natt),rep(0.05,natt)),
                                 hjust=0,cex=.8,col="gray14"),
                bg_params = list(fill=c(rep("white",natt)),col=c(rep("white",natt)),alpha = 0)),
      rowhead=list(fg_params=list(x=1, hjust=1, fontface="plain", fontfamily="Helvetica",cex=.8,col="gray14"),
                   bg_params = list(fill=c(rep("white",natt)),col=c(rep("white",natt)),alpha = 0)),padding = unit(c(0.8, .9), "lines"))

  pushViewport(viewport(x=.4,y=.5,height=.5))
  grid.table(t(attrib_table),theme=t1,cols = NULL)
```

![](load_single_catchment_files/figure-markdown_github/unnamed-chunk-3-1.png)

### 4) Graficar series de tiempo

Se puede cargar cualquiera de las series de tiempo descargadas del explorador camels-cl.

``` r
ts1 <- ts1[ts1$date>=date_ini & ts1$date<=date_end,]
ts2 <- ts2[ts2$date>=date_ini & ts2$date<=date_end,]
par(mfrow=c(2,1),
    oma=c(1, 2, 1, 1),
    mar=c(1, 3, 2, 1),
    mgp=c(2,0.5,0)
)
plot(ts1$date,t(ts1[,var_ts1]),
    type="l",col="royalblue1",cex.lab=1.3,lwd=1,
    panel.first = grid (NULL,NULL, lty = 1, lwd = .5, col = "lightgrey"),
    ylab=var_ts1,xlab="",xaxt = "n",cex.lab=.9,cex=.9,cex.axis=.8)
mtext(paste("Cuenca ", attrib$gauge_id, sep=""), line=.5,cex=.8,side=3)
axis.Date(1, ts1$date, format="%d/%m/%y",lwd=0.6,cex.axis=.8)

plot(ts2$date,t(ts2[,var_ts2]),
    type="l",col="royalblue1",cex.lab=1.3,lwd=1,
    panel.first = grid (NULL,NULL, lty = 1, lwd = .5, col = "lightgrey"),
    ylab=var_ts2,xlab="",xaxt = "n",cex.lab=.9,cex=.9,cex.axis=.8)
axis.Date(1, ts2$date, format="%d/%m/%y",lwd=0.6,cex.axis=.8)
```

![](load_single_catchment_files/figure-markdown_github/unnamed-chunk-4-1.png)
