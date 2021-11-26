# ProyGraficas

Proyecto en WebGL para la materia de Gráficas Computacionales. El proyecto consiste en una escena realizada en WebGL con la intención de simular el flujo de movilización de personas dentro del área metropolitana de Monterrey. Se considera:

	* El mapa base con el que se usara como fondo de la escena. 
	* Los movimientos tomados de un dataset externo que representen el desplazamiento. 

Véase /urbmobpy/ZMM_Urbmov.ipynb para más detalle

### Preprocesamiento del dataset

El dataset consiste en un conjunto de puntos con coordenadas en latitud y longitud en EPSG = 4326, con una estampa de tiempo en UTC. Cada punto representa una posición en un momento dado de un elemento identificado por el agent_id. Los datos cubren dos semanas, entre el 11 y el 25 de agosto.

Antes de pasar los datos a la capa de Deck, es necesario realizar un par de pasos de preprocesamiento:
* Pasar el utc_timestamp a una timestamp legible
* Concatenar latitud y longitud en un arreglo de coordenadas
* Con la finalidad de seleccionar el día con más puntos, se extrae la fecha en una columna separada.


El performance del equipo presenta un limitante por lo que elegí 100 de los agent_ids con más puntos durante el día con más puntos del dataset. Para ello tenemos que el día 13 cuenta con 167745 puntos. En promedio tenemos ~3773 puntos por agent_id. 
La capa de trayectoria de deck.gl usa un formato especifico, donde los puntos que forman un "path" deben de estar en un arreglo de las mismas dimensiones que las muestras de tiempo. Siguiendo la documentación [de la capa](https://deckgl.readthedocs.io/en/latest/gallery/trips_layer.html) se creo un dataframe para los 100 agent_ids con esas características

### Trips layer

El trips layer es una representación gráfica de un viaje. La capa toma el punto inicial y, de acuerdo a un arreglo de unidades temporales ordenado, va dibujando una línea que se desvanece. Entonces, en el momento dado el punto en t está en un color más sólido que los puntos en t-1, t-2.. , t-n. El parametro de esta "cola" se llama trail_length.

La capa puede ser renderizada de manera "estática" (el mapa aún puede moverse) o si se ajusta el parámetro de current time puede animarse.

En el contexto de pydeck (los bindings de deck.gl usados para el proyecto) una visualización está conformado por tres puntos: 
* Un objeto view (o ViewState) que representa la vista inicial del mapa y no puede ser actualizado.
* Una serie de capas. Véase: [este link](https://deckgl.readthedocs.io/en/latest/index.html) para mayor información
* Un objeto deck que junta todos estos. Este objeto puede ser renderizado a html o ejecutado como un widget de jupyter. 

Este último punto es la razón por la cuál el proyecto está siendo entregado por este medio. Las capas dentro de un jupyter widget si pueden ser actualizadas permitiendo animarlas.

Para la visualización se usaron 3 widgets de jupyter que proveen de interactividad:
* Un slider que permite ajustar la ventana de tiempo deseada
* Un botón de play que "reproduce" la animación del movimiento de los puntos
* Un cuadro de texto que muestra el tiempo en formato normal. 


### Conclusiones

Este proyecto me sirvió para entender como implementar bajo condiciones limitadas algún tipo de animación. Originalmente pensaba realizar el proyecto en react+javascript donde la librería de deck.gl tiene más funcionalidades. Pero al final por cuestiones de tiempo opté por usar los bindings para python. Esto me limitaba a desarrollar algo dentro del entorno de jupyter pero usando unos trucos con los widgets para crear callbacks y entendiendo la animación como un cambio en términos del tiempo de una serie de puntos pude logra desarrollar la visualización. Me hubiera gustado probarlo en un entorno con GPU para visualizar los 3700 identificadores únicos. Creo que hubiera sido más atractivo visualmente. Trataré de implementarlo en los siguientes días en react esperando mejorar el rendimiento; en esta iteración hay demasiadas capaz de por medio => deck - mapbox - jupyter - python.

También resta la pregunta si este tipo de tareas no se pudiera beneficiar de paralelismo implementado a manera de futuros en python dado que no todos los puntos se deben de dibujar de manera simultanea, podrían usarse hilos y no bloquear el intérprete. 