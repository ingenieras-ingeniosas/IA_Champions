link repo: https://github.com/ingenieras-ingeniosas/IA_Champions

# Predicción ganador UEFA Champions League 23-24

## Índice

- [Introducción](#introducción)
- [Metodología](#metodología)
  - [Recopilación de datos](#recopilación-de-datos)
    - [Extracción](#extracción)
    - [Estadísticas equipos](#estadísticas-equipos)
  - [Preprocesamiento](#preprocesamiento)
    - [Ponderaciones Ligas Nacionales](#ponderaciones-ligas-nacionales)
    - [Ponderaciones UCL](#ponderaciones-ucl)
    - [Dataset partidos](#dataset-partidos)
- [Análisis](#análisis)
  - [Modelo Regresión Lineal](#modelo-regresión-lineal)
  - [Modelo Red Neuronal](#modelo-red-neuronal)
    - [DNN 1: estadísticas futbolísticas](#dnn-1-estadísticas-futbolísticas)
    - [DNN 2: estadísticas históricas](#dnn-2-estadísticas-históricas)
- [Evaluación](#evaluación)
  - [Regresión Lineal](#regresión-lineal)
  - [DNN](#dnn)
- [Conclusiones](#conclusiones)

## Introducción
El objetivo de este proyecto era simple, haciendo uso de técnicas de Análisis de Datos y Machine Learning, predecir el equipo ganador del torneo de la UEFA Champions League (UCL) 2023/24. Con total libertad para elegir el planteamiento del análisis, los datos, los modelos a utilizar...

## Metodología

### Recopilación de datos
El vasto mundo del fútbol y las páginas de estadísticas futbolísticas es complicado. Hay muchas webs con mucha información y recopilar datos en grandes cantidades no es tarea fácil.

#### Extracción
Por suerte, dimos con la librería [ScraperFC](https://github.com/oseymour/ScraperFC) de **@ouseymour**, para el scrappeo de datos futbolísticos. Entre las diferentes webs de las que permite obtener datos, elegimos FBRef, ya que nos pareció la más completa en términos de las estadísticas que ofrece y la gran variedad de ligas internacionales que maneja.

Con el fin de predecir el ganador del torneo de la UCL 23/24, hemos tomado los siguientes datos: 

- Estadísticas de la UCL 2022/2023
- Estadísticas de la fase de grupos de la UCL 2022/2023
- Estadísticas de las ligas nacionales 2022/2023
- Estadísticas de las ligas nacionales 2023/2024 (hasta la fecha)

Decidimos coger los datos de las 2 últimas temporadas porque, en general, los equipos suelen mantener su plantilla relativamente constante durante este período.

#### Estadísticas equipos
Para obtener una visión completa de cada equipo, seleccionamos variables que cubren tanto aspectos de ataque como de defensa, así como otras variables relevantes. Las variables recopiladas para el análisis y su significado son las siguientes:

**Ataque**

- **Gls90**: Goles por 90 minutos.
- **Ast90**: Asistencias por 90 minutos.
- **G+A90**: Goles y asistencias combinados por 90 minutos.
- **G-PK90**: Goles no provenientes de penaltis por 90 minutos.
- **G+A-PK90**: Goles y asistencias no provenientes de penaltis por 90 minutos.
- **SCA90**: Acciones que conducen a una oportunidad de gol por 90 minutos.
- **GCA90**: Acciones que conducen a un gol por 90 minutos.
- **Sh90**: Disparos por 90 minutos.
- **SoT90**: Disparos a puerta por 90 minutos.
- **SoT%**: Porcentaje de disparos a puerta.
- **G/Sh**: Goles por disparo.
- **G/SoT**: Goles por disparo a puerta.

**Defensa**

- **GA90**: Goles concedidos por 90 minutos.
- **Save%**: Porcentaje de paradas realizadas por el portero.
- **SoTA90**: Disparos a puerta enfrentados por 90 minutos.

### Preprocesamiento

#### Ponderaciones Ligas Nacionales
Dado que cada liga europea presenta un nivel competitivo diferente, creímos necesario ponderar las estadísticas de los equipos en sus ligas nacionales, según el "nivel futbolístico" de la liga de procedencia:

- Las cinco principales ligas europeas Premiere League (Inglaterra), La Liga (España), Ligue 1 (Francia), Serie A (Italia) y Bundesliga (Alemania) se ponderaron con un factor de 1.
- Las ligas "de 2º categoría" serían la Eredivisie (Países Bajos), Superliga (Dinamarca) y Primeira Liga (Portugal), consideradas de un nivel ligeramente inferior, se ponderaron con un factor de 0.67.

#### Ponderaciones UCL
Para reflejar de manera precisa las diferencias de nivel entre los partidos de fase de grupos de la UCL y los enfrentamientos en fases más avanzadas del torneo, es esencial ponderar las estadísticas de los equipos. La mayoría de los partidos en las fases iniciales de la UCL son contra equipos significativamente inferiores, lo cual puede inflar las estadísticas de los equipos más fuertes. Para contrarrestar este efecto y lograr una representación más equilibrada, es necesario ajustar las estadísticas de la UCL añadiendo las estadísticas de las ligas nacionales de cada equipo.

La fórmula de ponderación utilizada para calcular las estadísticas combinadas es la siguiente:

\[ \text{Ponderación Final} = \frac{\left(\text{Est}_{\text{UCL}} + 0.5 \cdot \text{Est}_{\text{NAC}}\right)}{2} \]

Esta fórmula permite integrar las estadísticas nacionales, dándoles un peso moderado al multiplicarlas por 0.5, antes de combinarlas con las estadísticas de la UCL. Finalmente, el resultado se divide por 2 para obtener una ponderación equilibrada entre ambos conjuntos de datos. Esta metodología asegura que las estadísticas utilizadas en el análisis sean representativas del rendimiento real de los equipos, considerando tanto su desempeño en la UCL como en sus respectivas ligas nacionales.

#### Dataset partidos
Nuestro enfoque para predecir quién ganará el campeonato será simular los enfrentamientos, prediciendo los goles marcados por cada equipo, con ello, según quien gane, llegaremos al ganador de la UCL 23/24.

Para ello, entrenaremos los modelos con los partidos de la temporada completa 22/23 y la fase de grupos de la 23/24. Junto con las variables relativas al enfrentamiento, incluiremos las estadísticas de cada equipo, para que el modelo pueda aprender de ellas para predecir los goles.

Las variables del enfrentamiento son:

- **Nombre_Eq1**: equipo 1
- **Nombre_Eq2**: equipo 2
- **Gol_Match_Eq1**: goles equipo 1
- **Gol_Match_Eq2**: goles equipo 2
- **GANADOR**: nombre del equipo ganador

De esta forma el dataset nos queda con las variables:

![Variables](img/var-1.png)
![Variables](img/var-2.png)

## Análisis
Nuestro objetivo es predecir el número de goles que marca cada equipo en cada partido en función de sus estadísticas y las del adversario. Por lo tanto, al estar prediciendo una variable numérica, nuestros modelos serán de regresión.

Para simular el torneo, hemos hecho que en las fases de octavos, cuartos y semis haya ida y vuelta, para que el modelo pueda considerar el factor jugar en "casa". Es por esto que se muestran los goles acumulados, salvo en la final que es a partido único.

Los modelos que hemos utilizado son:

### Modelo Regresión Lineal

| Fase   | Equipo 1          | Gol Acum. Eq 1 | Equipo 2         | Gol Acum. Eq 2 |
|--------|-------------------|----------------|------------------|----------------|
| Octavos| PSG               | 3.612          | Real Sociedad    | 3.165          |
| Octavos| Copenhague        | 3.421          | Manchester City  | 5.581          |
| Octavos| Barcelona         | 3.647          | Napoli           | 2.308          |
| Octavos| Atlético Madrid   | 3.539          | Inter de Milán   | 3.313          |
| Octavos| Dortmund          | 3.819          | PSV              | 2.808          |
| Octavos| Bayern Múnich     | 3.186          | Lazio            | 2.166          |
| Octavos| Arsenal           | 4.346          | Porto            | 3.916          |
| Octavos| Real Madrid       | 4.161          | Leipzig          | 3.245          |
| Cuartos| Dortmund          | 2.995          | Atlético Madrid  | 3.4            |
| Cuartos| Barcelona         | 3.059          | PSG              | 2.941          |
| Cuartos| Arsenal           | 2.918          | Bayern Múnich    | 2.294          |
| Cuartos| Real Madrid       | 4.014          | Manchester City  | 4.259          |
| Semis  | Barcelona         | 2.835          | Atlético Madrid  | 3.144          |
| Semis  | Real Madrid       | 4.033          | Arsenal          | 3.476          |
| Final  | Atlético Madrid   | 2.339          | Real Madrid      | 2.387          |

#### Selección del modelo

Como en todos los proyectos de Machine Learning, probamos diferentes hiperparámetros y optimizadores para quedarnos con la mejor configuración. La regresión lineal utiliza la configuración básica, sin hiperparámetros relevantes a configurar.

Los hiperparámetros y optimizadores probados para las redes neuronales han sido:

- Optimizadores: Adam, SGD
- Número de capas: 1-3
- Número de neuronas: 32, 64, 128

Los resultados de los modelos con los mejores hiperparámetros y optimizadores se pueden observar en la sección de evaluación.

### Modelo Red Neuronal

Dado que existen numerosas variables que pueden influir en el resultado de un partido de fútbol y que la relación entre estas variables puede no ser lineal, decidimos explorar también el uso de redes neuronales. Una red neuronal puede capturar relaciones más complejas entre las variables y ofrecer mejores predicciones.

#### DNN 1: estadísticas futbolísticas

Para este modelo, hemos utilizado todas las estadísticas mencionadas anteriormente (tanto de ataque como de defensa) de ambos equipos, así como las variables del enfrentamiento.

#### DNN 2: estadísticas históricas

En este modelo, además de las estadísticas de la temporada actual, hemos incluido estadísticas históricas de los equipos, como el número de títulos de liga ganados en los últimos 10 años y el rendimiento en competiciones europeas.

## Evaluación

### Regresión Lineal
El modelo de regresión lineal mostró un desempeño decente, capturando algunas tendencias generales en los datos. Sin embargo, su capacidad para manejar relaciones no lineales y complejas es limitada, lo que puede afectar la precisión de las predicciones.

### DNN
Las redes neuronales, con su capacidad para capturar relaciones no lineales, mostraron un desempeño superior. A continuación, se muestran los resultados del modelo de regresión lineal y de las redes neuronales con sus respectivas métricas de evaluación, como el RMSE (Root Mean Square Error) y el MAE (Mean Absolute Error):

| Modelo                | RMSE | MAE  |
|-----------------------|------|------|
| Regresión Lineal      | 1.12 | 0.89 |
| DNN (futbolísticas)   | 0.95 | 0.76 |
| DNN (históricas)      | 0.88 | 0.70 |

## Conclusiones
En conclusión, hemos desarrollado y evaluado modelos de Machine Learning para predecir el ganador de la UCL 2023/24. Utilizando tanto la regresión lineal como redes neuronales, hemos considerado variables tanto futbolísticas como históricas. Las redes neuronales, en particular las que incluyen estadísticas históricas, mostraron un mejor desempeño en términos de precisión de predicción.

Esperamos que este enfoque pueda servir de base para futuros trabajos en el ámbito de las predicciones deportivas, destacando la importancia de la combinación de datos históricos y actuales para obtener mejores resultados.

