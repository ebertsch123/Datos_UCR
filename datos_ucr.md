---
title: "¿Cómo se usa el FEES en la UCR?"
author: "Asociación de Estudiantes de Química y Esteban Bertsch Aguilar"
date: "2024-08-20"
output: 
  html_document:
    keep_md: true
---
---

La lucha por el FEES ha creado una serie de narrativas para poner en duda los méritos de las Universidades Públicas. Los principales argumentos consisten en:

- Las universidades contratan demasiado personal, comparado a los estudiantes que tienen.
- Las universidades le otorgan salarios excesivos a su personal.
- La mayoría del presupuesto del FEES se va en esos salarios excesivos.

---

En este documento vamos a explorar las bases de datos de la U para ver varias cosas:

- ¿Cuántxs estudiantes entran a la U?
- ¿Cuánto personal se contrata en la U?
- ¿Cómo se distribuye el salario entre funcionarixs?


Para ello, vamos a usar las bases de datos de la UCR (disponibles en transparencia.ucr.ac.cr) y haremos un análisis **100% basado en datos**.



## **1. Estudiantes vs Funcionarixs**

Vamos a cargar las bases de datos de estudiantes matriculadxs y la planilla de la UCR.


```r
rm(list = ls())
library(tidyverse)
```

```
## Warning: package 'ggplot2' was built under R version 4.3.1
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.2     ✔ readr     2.1.4
## ✔ forcats   1.0.0     ✔ stringr   1.5.0
## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
## ✔ purrr     1.0.1     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

```r
#cargar tablas de estudiantes matriculados
estudiantes <- read.csv("https://transparencia.ucr.ac.cr/medios/documentos/2020/estudiantes-fi%CC%81sicos-matri%CC%81culados.csv",fileEncoding='latin1',check.names=F, sep = ';')
estudiantes <- replace(estudiantes, is.na(estudiantes),0)

n_estudiantes <- sum(estudiantes$`2019/I Ciclo/Estudiantes físicos`) #¿cuántos estudiantes matricularon en el primer semestre del 2019?




#cargar tabla de planilla de la UCR en abril 2024
funcionarios <- read.csv("https://transparencia.ucr.ac.cr/medios/documentos/2024/planilla-2024-04.csv",fileEncoding='latin1',check.names=F, sep = ';')



funcionarios$puesto <- ifelse(grepl('PROFESOR',funcionarios$`ï»¿PUESTO`) == TRUE,
                              'Docente',
                              'Administrativo')


docentes <- funcionarios %>% filter(puesto == 'Docente')
n_docentes <- nrow(docentes)

administrativos <- funcionarios %>% filter(puesto == 'Administrativo')
n_administrativos <- nrow(administrativos)
```



```r
df <- data.frame('Nombre' = c('Estudiantes', 'Docentes', 'Administrativos'),
                 'n' = c(n_estudiantes, n_docentes, n_administrativos))


ggplot(df) + 
  geom_col(aes(x = n, y = Nombre, fill = Nombre),
           color = 'black') + 
  theme(panel.background = element_blank(),
        panel.border = element_rect(fill = NA, color = 'black'),
        axis.text = element_text(size = 13),
        legend.position = 'none') + 
  labs(x = '', y = '') + 
  scale_fill_manual(values = c('red4', 'gold3', 'blue4')) + 
  annotate('text', x = 35000, y = 3, label = paste(n_estudiantes,'estudiantes'), color = 'white', size = 5) + 
  annotate('text', x = 13400, y = 2, label = paste(n_docentes,'docentes'), size = 5) + 
  annotate('text', x = 13000, y = 1, label = paste(n_administrativos,'administrativos'), size = 5)
```

![](README_figs/README-unnamed-chunk-2-1.png)<!-- -->

Esto significa que por cada estudiante, hay 0.12 docentes en la universidad.

Similarmente, por cada estudiante, hay 0.09 administrativos en la universidad.


---




