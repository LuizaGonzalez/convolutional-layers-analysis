# Exploración de las capas convolucionales a través de datos y experimentos

**Dataset elegido:** BananaLSD (Banana Leaf Spot Diseases Dataset)

**Kaggle:** https://www.kaggle.com/datasets/shifatearman/bananalsd

#### Justificación del dataset

El banano es uno de los cultivos más importantes del mundo. El dataset BananaLSD se hizo con fotos tomadas en campos de banano en Bangladesh, usando cámaras de celular en condiciones reales, es decir, con distinta luz, fondo y ángulo, como pasaría en la vida real. Un experto en enfermedades de plantas revisó cada foto y le puso su etiqueta correcta, así que los datos son confiables.

El dataset tiene 4 categorías: hoja sana y tres enfermedades del banano (Sigatoka, Cordana y Pestalotiopsis). Sirve bien para una red convolucional porque las enfermedades se ven como manchas que pueden aparecer en cualquier parte de la hoja. Las capas convolucionales son buenas justo para eso: reconocen un patrón sin importar en qué parte de la imagen esté.

Además, es un dataset pequeño y fácil de manejar: tiene 937 fotos originales, o 1600 si se usa la versión con imágenes generadas artificialmente. Todas las fotos tienen el mismo tamaño, así que no se necesita una computadora muy potente para trabajar con él.