# iccasbrazil
#Code for running matching analysis

## 1- Packages
library (survival)
library(optmatch)
library(MatchIt)
library(Zelig)
library(raster)
library(tidyverse)
library(spData)
library(spDataLarge)
library(sp)
library(rgdal)
library(Zelig)  

#################################################################################
setwd("doc")
dir()
dist_def_2005 <-raster ('amazonia_2005_reclass_proximity_AM_1km_albers.tif')
dist_def_2011 <-raster ('amazonia_2011_reclass_proximity_AMZ_1km_albers.tif')
prox_urb <-raster ('AreasUrbanizadas_neotropico_IBGE_e_NatEarth_proximity_AMZ_1km_albers.tif')
buff <-raster ('buffer_ucs_5km_AM_1km_albers.tif')
desmat <-raster ('Desmatamento_mapbiomas_2005_30porcento_AM_1km_albers.tif')
alag <-raster ('hand_wgs_AM_1km_albers.tif')
prox_hidro<-raster ('hidrovia_raster_proximity_AMZ_1km_albers.tif')
ti <-raster ('il_2005_reg_AM_1km_albers.tif')
alt <-raster ('MDE_AM_1km_albers.tif')
milico <-raster ('milico_AM_1km_albers.tif')
precip <-raster ('precipitacao_AM_1km_albers.tif')
quil<-raster ('quilombolas_LA_AM_1km_albers.tif')
prox_rod <-raster ('Rodovias_IBGE_proximity_AMZ_1km_albers.tif')
slope <-raster ('slope_AM_1km_albers.tif')
state <-raster ('states_wgs_AM_1km_albers.tif')
ucpi <-raster ('ucpi_05_AM_1km_albers.tif')
ucus <-raster ('ucus_05_AM_1km_albers.tif')
ucus_no_apa <-raster ('ucus_05_noapa_AM_1km_albers.tif')
veg_2011 <-raster ('Vegetacao_2011_AM_1km_albers.tif')


## 2- stack layers
raster_all <- stack(dist_def_2005, dist_def_2011, prox_urb, buff, desmat, alag, prox_hidro,
                    ti, alt, milico, precip,quil, prox_rod, slope, state, ucpi,
                    ucus, ucus_no_apa,veg_2011)
plot(raster_all)
raster_all                  

## 3- Create Matrix on stack:as.data.frame
data_all <- as.data.frame(raster_all, row.names = NULL, xy=TRUE, na.rm = T)
write.csv(data_all, file = "data_all_incompleto.csv")
count(data_all)
id <- c(1:4284587) 
sant_dataframe <- cbind(id, data_all)[, c(1:22)] # insert a column

## 4- Saving dataframe
write.csv(sant_dataframe, file = "stack_dataframe.csv")
sant_dataframe= read.csv("stack_dataframe.csv")
head (sant_dataframe)

## 5 - subsetting
dataframe=subset(sant_dataframe, states_wgs_AM_1km_albers == 13)  
head (dataframe)

dataframe_sembuff=subset(dataframe, buffer_ucs_5km_AM_1km_albers == 0) 
head (dataframe_sembuff)

dataframe_sembuff_desm=subset(dataframe_sembuff, Desmatamento_mapbiomas_2005_30porcento_AM_1km_albers == 0)  
head (dataframe_sembuff_desm)

dataframe_sembuff_desm_mili=subset(dataframe_sembuff_desm, milico_AM_1km_albers == 0) 
head (dataframe_sembuff_desm_mili)

dataframe_sembuff_desm_mili_quil=subset(dataframe_sembuff_desm_mili, quilombolas_LA_AM_1km_albers == 0) 
head (dataframe_sembuff_desm_mili_quil)

dataframe_sembuff_desm_mili_quil_ucpi=subset(dataframe_sembuff_desm_mili_quil,  ucpi_05_AM_1km_albers == 0) 
head (dataframe_sembuff_desm_mili_quil_ucpi)

dataframe_sembuff_desm_mili_quil_ucpi_ucus=subset(dataframe_sembuff_desm_mili_quil_ucpi, ucus_05_AM_1km_albers == 0) 
head (dataframe_TO_sembuff_desm_mili_quil_ucpi_ucus)

dataframe_sembuff_desm_mili_quil_ucpi_ucus_ucusnoapa=subset(dataframe_sembuff_desm_mili_quil_ucpi_ucus, ucus_05_noapa_AM_1km_albers == 0) 
head (dataframe_sembuff_desm_mili_quil_ucpi_ucus_ucusnoapa)
#46914390 pixel
count(dataframe_sembuff_desm_mili_quil_ucpi_ucus_ucusnoapa)

dataframe_sembuff_ti<-dataframe_sembuff_desm_mili_quil_ucpi_ucus_ucusnoapa[,-c(13,15,19,20,21)]
head(dataframe_sembuff_ti)


## 6- matching 
start= Sys.time()
matched_ti <- matchit(dataframe_sembuff_ti$il_2005_reg_AM_1km_albers ~ dataframe_sembuff_ti$amazonia_2005_reclass_proximity_AM_1km_albers + 
                       dataframe_sembuff_ti$amazonia_2011_reclass_proximity_AMZ_1km_albers +
                       dataframe_sembuff_ti$AreasUrbanizadas_neotropico_IBGE_e_NatEarth_proximity_AMZ_1km_albers +
                       dataframe_sembuff_ti$Desmatamento_mapbiomas_2005_30porcento_AM_1km_albers + 
                       dataframe_sembuff_ti$hand_wgs_AM_1km_albers + 
                       dataframe_sembuff_ti$hidrovia_raster_proximity_AMZ_1km_albers+
                       dataframe_sembuff_ti$MDE_AM_1km_albers+
                       dataframe_sembuff_ti$precipitacao_AM_1km_albers + 
                       dataframe_sembuff_ti$Rodovias_IBGE_proximity_AMZ_1km_albers  + 
                       dataframe_sembuff_ti$slope_AM_1km_albers + 
                       dataframe_sembuff_ti$states_wgs_AM_1km_albers + 
                       dataframe_sembuff_ti$Vegetacao_2011_AM_1km_albers, 
                     method="nearest", replace=F, data = dataframe_sembuff_ti) 
print(Sys.time()-start)

## 7- Results 
plot(matched_ti)
summary(matched_ti)
print(matched_ti)

## 8- new matrix and export
matched2 <- match.data(matched_replace)
write.csv(matched2, file = "/dados/Helena/dados/matched_nearest_replace.csv")


