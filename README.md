---
title: "¿Cómo se usa el FEES en la UCR?"
author: "Asociación de Estudiantes de Química y Esteban Bertsch Aguilar"
date: "2024-08-20"
output: 
  html_document:
    keep_md: true
---

# **¿Cómo se usa el FEES en la UCR?**

*Asociación de Estudiantes de Química*

*Esteban Bertsch-Aguilar*

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

#cargar tablas de estudiantes matriculados
estudiantes <- read.csv("https://transparencia.ucr.ac.cr/medios/documentos/2020/estudiantes-fi%CC%81sicos-matri%CC%81culados.csv",fileEncoding='latin1',check.names=F, sep = ';')
estudiantes <- replace(estudiantes, is.na(estudiantes),0)

n_estudiantes <- sum(estudiantes$`2020/I Ciclo/Estudiantes físicos`) #¿cuántos estudiantes matricularon en el primer semestre del 2020?




#cargar tabla de planilla de la UCR en abril 2024
funcionarios <- read.csv("https://transparencia.ucr.ac.cr/medios/documentos/2024/planilla-2024-05.csv", sep = ';')



funcionarios$puesto <- ifelse(grepl('PROFESOR',funcionarios$PUESTO) == TRUE,
                              'Docente',
                              'Administrativo')


docentes <- funcionarios %>% filter(puesto == 'Docente')
n_docentes <- nrow(docentes)

administrativos <- funcionarios %>% filter(puesto == 'Administrativo')
n_administrativos <- nrow(administrativos)



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

![](README_figs/README-unnamed-chunk-1-1.png)<!-- -->


Esto significa que por cada **estudiante matriculado**, hay **0.12 docentes** en la universidad (*o ~8 estudiantes por cada docente*).

Similarmente, por cada **estudiante matriculado**, hay **0.09 administrativos** en la universidad (*o ~11 estudiantes por cada administrativo*).


---

## **2. Hablemos de salarios**

Una de las principales críticas hacia las universidades públicas consiste en el salario de sus funcionarios. La opinión pública prevalece que los salarios dentro de las universidades son excesivamente altos.

La UCR brinda una base de datos cada mes con todos los puestos y salarios de toda la planilla de la universidad. Indaguemos en la realidad salarial del país. Para ello, vamos a usar la planilla a mitad de semestre, en abril 2024 (*para que estén incluidos los profes interinos, quienes se les quita el nombramiento cada final de semestre*).


```r
#función para encontrar la moda
get_mode <- function(x) {
  unique_x <- unique(x)
  unique_x[which.max(tabulate(match(x, unique_x)))]
}

promedio <- round(mean(funcionarios$SALARIO),0)
moda <- round(get_mode(funcionarios$SALARIO),0)


ggplot(funcionarios) + 
  geom_histogram(aes(x = SALARIO), color = 'black', fill = 'cyan4', bins = 40) + 
  geom_vline(xintercept = promedio, linewidth = 1) + 
  geom_vline(xintercept = moda, linewidth = 1) + 
  geom_vline(xintercept = 337626, linewidth = 1.2, color = 'blue4') +
  labs(x = 'Salarios', y = 'Cantidad') + 
  theme(panel.background = element_blank(),
        panel.border = element_rect(fill = NA, color = 'black'),
        text = element_text(size = 14),
        legend.position = 'none')
```

![](README_figs/README-unnamed-chunk-2-1.png)<!-- -->

Este es el salario promedio en la UCR:


```r
print(paste0("₡",promedio))
```

```
## [1] "₡1219153"
```

Sin embargo, el salario más frecuente dentro de la UCR es este:


```r
print(paste0("₡",moda ))
```

```
## [1] "₡212228"
```

Como referencia, comparémoslo con el salario mínimo que recibe un docente en la Universidad de Buenos Aires (https://aduba.org.ar/tabla-salarial/).:


```r
print(paste0("~ ₡337626"))
```

```
## [1] "~ ₡337626"
```


Con ello, podemos ver una realidad más objetiva con los salarios asignados en la U. ¿Por qué el salario más frecuente es más bajo? Una gran parte de funcionarios de la UCR (approx. 1000) no están nombrados a tiempo completo. Veamos con más detalle cómo se distribuyen esas pagas dependiendo del puesto.

Veamos cómo se distribuyen los salarios de cada funcionario dependiendo de su puesto:



```r
#separación más específica por puesto

for(i in 1:nrow(funcionarios)){
  if(grepl('INTERINO',funcionarios$PUESTO[i]) == TRUE){
    funcionarios$puesto[i] <- 'Docente interino'
  } else if(grepl('ADJUNTO',funcionarios$PUESTO[i]) == TRUE){
    funcionarios$puesto[i] <- 'Docente adjunto'
  } else if(grepl('INSTRUCTOR',funcionarios$PUESTO[i]) == TRUE){
    funcionarios$puesto[i] <- 'Docente instructor'
  } else if(grepl('ASOCIADO',funcionarios$PUESTO[i]) == TRUE){
    funcionarios$puesto[i] <- 'Docente asociado'
  } else if(grepl('CATEDRATICO',funcionarios$PUESTO[i]) == TRUE){
    funcionarios$puesto[i] <- 'Docente catedrático'
  } else {funcionarios$puesto[i] <- 'Administrativo'}
}


funcionarios$salarios2 <- NA

for(i in 1:nrow(funcionarios)){
  if(funcionarios$SALARIO[i] < 1e6){
    funcionarios$salarios2[i] <- '< 1 millón'
  } else if(funcionarios$SALARIO[i] >= 1e6 & funcionarios$SALARIO[i] < 2e6) {
    funcionarios$salarios2[i] <- 'entre 1 y 2 millones'
  } else if(funcionarios$SALARIO[i] >= 2e6 & funcionarios$SALARIO[i] < 3e6) {
    funcionarios$salarios2[i] <- 'entre 2 y 3 millones'
  } else if(funcionarios$SALARIO[i] >= 3e6 & funcionarios$SALARIO[i] < 4e6) {
    funcionarios$salarios2[i] <- 'entre 3 y 4 millones'
  } else if(funcionarios$SALARIO[i] >= 4e6) {
    funcionarios$salarios2[i] <- '> 4 millones'
  }
}


funcionarios$salarios2 <- factor(funcionarios$salarios2, levels = c('< 1 millón','entre 1 y 2 millones','entre 2 y 3 millones','entre 3 y 4 millones','> 4 millones'))

n_salarios <- table(funcionarios$salarios2)


ggplot(funcionarios) + 
  geom_bar(aes(y = salarios2, fill = puesto), color = 'black') + 
  theme(panel.background = element_blank(),
        panel.border = element_rect(fill = NA, color = 'black'),
        text = element_text(size = 14),
        legend.position = 'bottom') + 
  labs(y = '', x = 'Cantidad') +
  scale_fill_manual('Puesto', values = c('#D55E00', '#0072B2', '#F0E442', '#009E73', '#CC79A7', '#56B4E9')) + 
  annotate('text', x = 5100, y = 1, label = n_salarios[[1]], size = 4) + 
  annotate('text', x = 3300, y = 2, label = n_salarios[[2]], size = 4) + 
  annotate('text', x = 1400, y = 3, label = n_salarios[[3]], size = 4) + 
  annotate('text', x = 600, y = 4, label = n_salarios[[4]], size = 4) + 
  annotate('text', x = 400, y = 5, label = n_salarios[[5]], size = 4)
```

![](README_figs/README-unnamed-chunk-6-1.png)<!-- -->

---

## **3. ¿Cuánto del FEES es realmente esto?**

En 2024, la UCR recibió ₡291317.11 millones de su corte del FEES (https://semanariouniversidad.com/universitarias/asi-quedo-distribuido-el-fees-de-2024/). Con esta información, vamos a analizar varios casos:

- ¿Cuánto % del corte de la UCR consisten estos salarios anuales?
- Vamos a despedir a todas las personas que ganen 4 millones o más. ¿Cuánto cambia ese porcentaje?
- Vamos a hacer un tope salarial de ₡2 millones. ¿Cuánto cambia ese porcentaje?
- Vamos a hacer un tope salarial de ₡1 millón. ¿Cuánto cambia ese porcentaje?



```r
FEES <- 291317.11*1e6 #corte del FEES para la UCR

funcionarios$despedidos <- ifelse(funcionarios$SALARIO > 4e6, 0, funcionarios$SALARIO)
funcionarios$corte2M <- ifelse(funcionarios$SALARIO > 2e6, 2e6, funcionarios$SALARIO)
funcionarios$corte1M <- ifelse(funcionarios$SALARIO > 1e6, 1e6, funcionarios$SALARIO)

porcentaje <- c()

for(i in c(2,9:11)){
  porcentaje <- c(porcentaje, 100*(sum(funcionarios[,i])*12)/FEES)
}

pcts <- data.frame('Pruebas' = relevel(as.factor(c('Planilla normal', 'Despidiendo salarios >4 millones', 'Tope 2 millones', 'Tope 1 millón')),'Planilla normal'),
                   'Porcentajes' = porcentaje)


ggplot(pcts) + 
  geom_col(aes(x = Porcentajes, y = Pruebas, fill = Porcentajes), color = 'black') + 
  theme(panel.background = element_blank(),
        panel.border = element_rect(fill = NA, color = 'black'),
        text = element_text(size = 14),
        legend.position = 'none') + 
  labs(y = '', x = 'Porcentaje (%)') + 
  scale_fill_gradient(low = '#0072B2', high = '#F0E442', limits = c(40,55))
```

![](README_figs/README-unnamed-chunk-7-1.png)<!-- -->


Y, en estos casos hipotéticos estamos asumiendo que se le está pagando a todo el personal interino durante los 12 meses del año (*lo cual no ocurre*).
