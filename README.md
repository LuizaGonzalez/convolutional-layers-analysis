# Exploración de las capas convolucionales a través de datos y experimentos
 
**Dataset elegido:** BananaLSD (Banana Leaf Spot Diseases Dataset)
 
**Kaggle:** https://www.kaggle.com/datasets/shifatearman/bananalsd
 
## Descripción del problema
 
El banano es uno de los cultivos más importantes del mundo. Las enfermedades en las hojas de banano bajan la producción y, si no se detectan a tiempo, pueden generar pérdidas grandes para los productores. En este proyecto se usa una red neuronal convolucional (CNN) para clasificar fotos de hojas de banano en 4 categorías, sana y 3 enfermedades, comparando el resultado contra una red totalmente conectada (modelo base), y probando cómo distintas decisiones de arquitectura afectan el aprendizaje.
 
## Descripción del dataset
 
El banano es uno de los cultivos más importantes del mundo. El dataset BananaLSD se hizo con fotos tomadas en campos de banano en Bangladesh, usando cámaras de celular en condiciones reales, es decir, con distinta luz, fondo y ángulo, como pasaría en la vida real. Un experto en enfermedades de plantas revisó cada foto y le puso su etiqueta correcta, así que los datos son confiables.
 
El dataset tiene 4 categorías: hoja sana y tres enfermedades del banano (Sigatoka, Cordana y Pestalotiopsis). Sirve bien para una red convolucional porque las enfermedades se ven como manchas que pueden aparecer en cualquier parte de la hoja. Las capas convolucionales son buenas justo para eso: reconocen un patrón sin importar en qué parte de la imagen esté.
 
Además, es un dataset pequeño y fácil de manejar: tiene 937 fotos originales, o 1600 si se usa la versión con imágenes generadas artificialmente. Todas las fotos tienen el mismo tamaño, así que no se necesita una computadora muy potente para trabajar con él.
 
| Dato | Valor |
|---|---|
| Total de imágenes (conjunto original) | 937 |
| Clases | 4: cordana, healthy, pestalotiopsis, sigatoka |
| Resolución original | 224x224 px |
| Canales | RGB (3) |
| Distribución de clases | sigatoka 50.5%, pestalotiopsis 18.5%, cordana 17.3%, healthy 13.8% (desbalanceado) |

## Diagrama de la arquitectura
![Arquitectura](/imagenes/arquitecturaCNN.drawio.png)
## Resultados experimentales
 
### Modelo base vs. CNN
 
| Métrica | Baseline (Dense) | CNN | Diferencia |
|---|---|---|---|
| Parámetros totales | 3,148,004 | 2,102,564 | 33% menos |
| Accuracy entrenamiento | 70.7% | 99.5% | — |
| Accuracy validación | 57.1% (inestable) | 80.0% | +23 pts |
| Accuracy test | 64.1% | 81.7% | +17.6 pts |
| Test loss | 0.9178 | 0.7648 | menor (mejor) |
 
El modelo base mostró sobreajuste marcado: la brecha entre accuracy de entrenamiento y validación crece con las épocas, y la validación no mejora de forma estable. Esto pasa porque al aplanar la imagen desde el inicio, el modelo pierde la relación entre píxeles vecinos y termina con 3.1 millones de parámetros, la mayoría en la primera capa densa.
 
La CNN generaliza bastante mejor, aunque tampoco está libre de sobreajuste (brecha de ~20 puntos entre train y validación), sobre todo porque la capa densa después del aplanado sigue siendo grande.

### Experimento controlado: profundidad de la capa convolucional
 
Comparé 3 arquitecturas idénticas en todo, excepto en el número de bloques convolucionales (1, 2 y 3), manteniendo fijo el resto.
 
| Profundidad | Train accuracy | Val accuracy | Test accuracy | Test loss |
|---|---|---|---|---|
| 1 capa | 99.4% | 75.7% | 77.5% | 0.65 |
| 2 capas | 98.0% | 75.0% | 70.4% | 0.87 |
| 3 capas | 97.9% | 75.7% | 77.5% | 0.93 |
 
Aumentar la profundidad de 1 a 3 capas no mejora el accuracy con este dataset, y la arquitectura más profunda (3 capas) resultó ser la más inestable durante el entrenamiento. Con solo ~655 imágenes de entrenamiento, el dataset parece no ser lo suficientemente grande como para que una red más profunda aproveche su capacidad extra de forma estable. Agregar más capas aquí no se tradujo en mejor generalización, solo en un modelo más costoso y más impredecible entre corridas.

##### **Interpretación**

**¿Por qué las capas convolucionales superaron (o no) al modelo base?**

Las capas convolucionales superaron al modelo base: el baseline llegó a 64.1% de accuracy en test, mientras que la CNN llegó a 81.7%. debido a que la convolución no aplana la imagen de entrada, mantiene la relación espacial entre píxeles vecinos, lo cual le permite reconocer patrones como manchas sin importar en qué parte de la hoja aparezcan. Mientras que en el modelo base, aplanaba la imagen desde el inicio, tratando cada píxel como un dato suelto, sin relación con sus vecinos, esto generaba muchísimos parámetros (3.1 millones) concentrados en la primera capa, y se presentaba el sobreajuste que vimos en las curvas de validación inestables. En la CNN, aplanar solo ocurre al final, después de que las capas convolucionales ya redujeron y resumieron la imagen en un mapa de características más pequeño, lo que ayuda a que el modelo generalice mejor.

**¿Qué sesgo inductivo introduce la convolución?**

La convolución ya trae dos supuestos, antes de ver algun dato. El primero es que los patrones importantes de una imagen están formados por píxeles vecinos, no sueltos, por eso el filtro solo mira un pedacito pequeño a la vez, no la imagen completa.

El segundo es que ese mismo filtro se repite por toda la imagen, así que si aprende a reconocer una mancha en una esquina, también la reconoce si aparece en el centro de otra foto. O sea, no importa en qué parte esté, el modelo igual la detecta.

Como las manchas de sigatoka, cordana o pestalotiopsis pueden salir en cualquier parte de la hoja, esto le favorece a mi dataset. El modelo base, en cambio, no tiene esa ventaja, mira cada píxel por separado, sin saber que están relacionados entre sí. Entonces le tocaba aprender eso solo, a punta de datos, y con apenas 655 imágenes no le alcanzó.

Esto también se ve reflejado en decisiones concretas de mi arquitectura: usar MaxPooling después de cada bloque convolucional redujo el tamaño del mapa de características antes de llegar al Flatten (de 128x128 a 32x32), lo cual evitó que la capa densa final tuviera tantos parámetros como en el baseline.

En mi arquitectura, esto se ve en el kernel de 3x3 que elegí: es lo suficientemente pequeño como para asumir que el patrón de una mancha está en un grupo compacto de píxeles vecinos, y ese mismo filtro se reutiliza en toda la imagen de 128x128.

**¿En qué tipo de problemas no sería apropiada la convolución?**

La convolución deja de tener sentido cuando los datos no tienen esa estructura de "vecindad" como veniamos viendo en las preguntas anteriores. Por ejemplo, en el ejercicio pasado de enfermedades cardiacas con columnas de edad, presión y colesterol, cada columna significa algo fijo y distinto, no tiene sentido que un filtro deslizante busque el mismo patrón entre "edad y presión" que entre "presión y colesterol". Ahí es mejor una red densa, donde cada columna se trata por separado.

También falla en cosas donde lo que importa es la relación entre datos que están muy separados entre sí, no entre vecinos cercanos. Por ejemplo, si quiero predecir si alguien va a comprar algo según sus compras de los últimos 6 meses, lo que compró el primer mes puede ser tan importante como lo que compró la semana pasada, no importa que estén lejos en el tiempo. Aqui la convolución no ayuda tanto.

En resumen, la convolución funciona cuando la cercanía entre valores tiene significado (como en imágenes), pero no cuando cada valor tiene un significado fijo e independiente de su posición.