# Análisis: precios al consumidor en Chile, 2026

## Contexto

ODEPA mide cada semana el precio de alimentos de la canasta familiar en distintos puntos de venta del país. Este análisis usa el archivo de 2026, que llega hasta la semana 47 (20 de noviembre). Quise responder cuatro preguntas: qué tan confiables son los datos, qué determina el precio y sus cambios, si el calendario deja huella y si los productos se pueden agrupar por su comportamiento anual.

## 1. Los datos

El archivo tiene 292.566 registros y 15 columnas, con 9 regiones, 41 sectores, 7 grupos de productos y 271 productos. No hay valores nulos ni filas duplicadas. En ninguna fila el precio mínimo supera al máximo y el promedio siempre queda entre ambos.

Hay dos cosas que conviene saber antes de usarlos:

- Los precios vienen con coma decimal y el archivo tiene una marca de codificación (BOM) al inicio.
- Hay 19 unidades de medida distintas (kilo, unidad, bandeja, caja, litro y otras). Comparar precios entre productos exige separar por unidad.

La muestra no está repartida de forma pareja. La Región Metropolitana, Valparaíso y La Araucanía concentran el 56% de los registros, y la mitad proviene de supermercados. Los resultados reflejan sobre todo esas zonas y ese canal.

## 2. Precio mínimo, máximo y promedio

En niveles, los tres precios son casi idénticos (correlaciones entre 0,98 y 0,997 usando logaritmos). Por eso predecir uno a partir de otro no tiene valor analítico.

Las variaciones semanales sí se diferencian: el mínimo y el máximo se correlacionan apenas 0,32, y cada uno se relaciona con el promedio (0,73 y 0,81). La semana del año no se correlaciona con ningún precio, así que no hay una tendencia común de alza o baja.

## 3. Qué variables explican los precios

Entrené modelos XGBoost con región, sector, tipo de punto, grupo, producto, unidad y semana, y medí la importancia de cada variable por permutación.

- **Nivel del precio:** el modelo acierta casi todo (R² entre 0,986 y 0,991). La variable dominante es el producto. El tipo de punto y la unidad aportan algo menor.
- **Variación semanal:** entrenando con las semanas 1 a 38 y probando en la 39 a 47, el R² es negativo (entre -0,15 y -0,31). Con estas variables no se puede anticipar si un precio sube o baja la semana siguiente.

La conclusión práctica es que explicar los movimientos de corto plazo requeriría información externa, como clima, tipo de cambio o combustibles.

Entre grupos, frutas y hortalizas son las más volátiles (variación semanal típica cercana al 13%), mientras que pan, lácteos y abarrotes se mueven entre 5% y 6%. Por tipo de punto, la feria libre es la más volátil (14%) y la panadería la más estable (6%).

## 4. Calendario

Marqué las semanas con Año Nuevo, Semana Santa, Día del Trabajo, Glorias Navales, Fiestas Patrias, Todos los Santos y vacaciones de invierno, y comparé la variación semanal de cada grupo en esas semanas contra el resto del año.

- **Semana Santa (semana 14):** lácteos y huevos suben 12,5% en esa semana, frente a -0,4% el resto del año. El alza viene del huevo de color, con aumentos de 25% a 75% según la calidad.
- **Todos los Santos:** las frutas suben 3,5% y las hortalizas bajan 2%.
- **Fiestas Patrias:** las hortalizas bajan 2% y los abarrotes 1%.
- **Vacaciones de invierno:** los abarrotes suben 4,7% y los lácteos bajan 2,4%.

Esto debe leerse con cautela. Cada evento corresponde a una sola semana de un solo año, y los valores p que entrega el cuaderno se calculan tratando cada producto como una observación independiente, lo que los hace más pequeños de lo que corresponde. El caso de Semana Santa es nítido; el resto no se puede separar con confianza de la estacionalidad propia de cada producto.

## 5. Estacionalidad mensual

Las hortalizas son las que más cambian durante el año: están entre 9% y 10% bajo su promedio anual en enero y febrero, y llegan a 15% sobre su promedio en octubre. Las frutas hacen casi lo contrario, con el mínimo en julio y alzas hacia noviembre. Pan, abarrotes y carnes tienen movimientos más acotados.

## 6. Clustering de productos

Calculé el precio mediano nacional de cada producto y unidad por semana, conservé las 206 series con al menos 40 semanas, y las transformé para que solo quedara la forma de la curva (logaritmo, centrado y escalado). Con K-means, el coeficiente de silueta fue más alto con tres grupos (0,263), frente a 0,23 con dos y 0,24 con cuatro.

| Grupo | Productos | Composición principal | Comportamiento |
|---|---|---|---|
| 1 | 54 | Lácteos y huevos (35), frutas (14) | Máximo en la semana 21, mínimo en la 5; salto en Semana Santa; el más volátil |
| 2 | 60 | Abarrotes (38), frutas (9) | Máximo en la semana 1, mínimo en la 29; baja durante el año |
| 3 | 92 | Hortalizas (39), carne bovina (22), frutas (10) | Máximo en la semana 41, mínimo en la 4; el más estable |

La silueta de 0,26 indica estructura real pero débil: hay productos en el límite entre grupos, y los resultados sirven para describir, no para clasificar con exactitud.

## Limitaciones y siguientes pasos

- Es un solo año, por lo que no puedo separar el efecto de un evento del de la temporada.
- No incorporé variables externas (clima, dólar, combustibles), que probablemente explican buena parte de las variaciones semanales.
- Los precios medianos nacionales ocultan diferencias regionales.

Como siguiente paso, sumaría los años 2024 y 2025 que ODEPA también publica, para confirmar los patrones estacionales y los efectos del calendario, y cruzaría los precios con el tipo de cambio.
