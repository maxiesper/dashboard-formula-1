# dashboard-formula-1
"Dashboard de Power BI sobre 75 años de Fórmula 1, con relaciones por ID y datos validados contra la historia real.

# Dashboard F1: reconstruir desde el error

Análisis de 75 años de Fórmula 1 (1950-2024) en Power BI: 1125 carreras, 861 pilotos, 212 constructores, 77 circuitos.

**[Ver el dashboard en vivo](https://app.powerbi.com/view?r=eyJrIjoiODU1NWMyZjYtY2E4NC00OWExLTg1NzEtMzUxMDQyY2Q1YzU4IiwidCI6ImZkM2FmZTllLTY0OGYtNDcxYy1iYTFmLTlmODhjZGI1MDc3ZCIsImMiOjR9)**

---

## Por qué empecé de cero

Tenía una primera versión de este dashboard, "Portfolio F1", y la abandoné. No por gusto: cuando la revisé en detalle encontré que las rutas a los CSV apuntaban a una carpeta que ya no existía, y que una tabla calculada que había armado (`race_wins`) relacionaba pilotos por nombre completo en vez de por ID. El resultado era una locura: Adolf Brudes, un piloto que corrió una sola carrera en toda su vida, en 1932, y salió último, me figuraba con 1116 victorias.

Ahí entendí que no tenía sentido parchar nada. Había que tirar el archivo y volver a armarlo desde cero, con una regla que me impuse desde el día uno: ninguna relación se arma por nombre. Todo por ID, sin excepción.

## Cómo lo encaré

Catorce tablas, todas relacionadas por sus claves numéricas (`driverId`, `raceId`, `constructorId`, y demás), nunca por texto. Y cada medida que armé la chequeé contra un dato real de la historia de la F1 antes de darla por buena. Si una medida me decía que Verstappen sacó 575 puntos en 2023 con 19 victorias, tenía que coincidir con lo que pasó de verdad, no solo "sonar bien".

Ese cruce constante entre lo que decía mi modelo y lo que dice la historia real terminó siendo la herramienta más efectiva que tuve para encontrar errores. Prácticamente todos los bugs importantes que corregí en este proyecto los agarré así: por un número que no cerraba con lo que yo ya sabía de F1.

## Los bugs que me fui encontrando

**Una relación que no debía existir.** Power BI detectó automáticamente una relación entre `sprint_results` y `results` por una coincidencia de nombre de columna que no tenía nada que ver una con otra. El mismo tipo de error, en el fondo, que me había dado a Adolf Brudes con 1116 victorias.

**El filtro que no viajaba.** Tenía varias medidas (campeón de temporada, campeón de constructores, primera y última temporada de cada piloto) que me daban en blanco para todos los años. La causa fue una relación de un solo sentido: si filtrás el lado "muchos" de una relación 1 a muchos, ese filtro no se propaga solo hacia el lado "1". Lo diagnostiqué armando medidas de prueba una atrás de otra hasta encontrar exactamente dónde se cortaba la cadena, en vez de ponerme a probar arreglos al azar.

**El punto decimal que desaparecía.** Tenía columnas de latitud y longitud de los circuitos, y de puntos de los pilotos, con valores imposibles: un piloto con 814 puntos en una sola carrera, un circuito en latitud -378497. Cuando fui a buscar la causa real, encontré que había un paso temprano en Power Query que convertía esas columnas a número entero antes de que cualquier arreglo posterior con configuración regional pudiera hacer algo. Para cuando yo intentaba corregir el separador decimal, el punto ya no estaba en ningún lado: se había perdido varios pasos antes, en un lugar que a simple vista no se notaba.

**Un campeón que murió antes de terminar la temporada.** Armé una medida para comparar si el campeón de Pilotos de cada año había corrido para el mismo equipo que se quedó con el título de Constructores. Dos años me daban "no coinciden" cuando en realidad sí coincidían: 1961 (Phil Hill, campeón con Ferrari, que también ganó Constructores) y 1970 (Jochen Rindt, campeón con Lotus, que también ganó Constructores). La causa: mi medida buscaba el equipo del campeón en la última carrera del calendario, asumiendo que había corrido esa carrera. Pero Rindt murió en Monza en septiembre de 1970 —hasta el día de hoy es el único campeón póstumo de toda la historia de la F1— y no llegó a correr las últimas tres fechas de esa temporada. Tuve que corregir la medida para que buscara la última carrera que el piloto realmente corrió, no la última del calendario a secas.

**Puntos que no eran un error.** Después de esa corrección me quedaron tres valores raros: 50, 36 y 30 puntos en una sola carrera. Antes de tocarlos, los rastreé, y los tres pertenecían a la misma carrera: el Gran Premio de Abu Dabi 2014, la única vez en la historia de la F1 que se usó una regla de puntos dobles para cerrar la temporada. No era un bug mío. Era historia real, y la dejé documentada así en vez de borrarla como si fuera un error.

## Las siete páginas

| Página | Hallazgo |
|---|---|
| **Conteo General** | El panorama completo: 1125 carreras, 861 pilotos, 212 constructores, 77 circuitos, 75 temporadas. |
| **Pilotos** | 103 de los 158 pilotos "estadounidenses" de la F1 nunca corrieron un Gran Premio real — solo las 500 Millas de Indianápolis, que puntuó para el Mundial entre 1950 y 1960. Por eso EE.UU., segundo país en cantidad de pilotos, cae al puesto 12 en victorias. |
| **Ficha de Piloto** | Perfil individual por piloto: foto, nacionalidad, fecha de nacimiento, edad de debut y evolución de puntos por temporada. |
| **Ganadores** | 115 ganadores distintos en 75 años de carreras. |
| **Constructores** | Solo 47 de los 212 equipos que pasaron por la F1 ganaron alguna vez una carrera. Ferrari solo se quedó con 249 de las 1125 victorias: casi 1 de cada 4. |
| **Campeones por Temporada** | De 67 temporadas con título de Constructores, en 12 (18%) el campeón de Pilotos no corrió para el equipo campeón. Pasó en 1958 (el primer año) y dos veces en las últimas cuatro temporadas: 2021 y 2024, las dos con Verstappen de por medio. |
| **Circuitos** | Monza estuvo en el calendario en 74 de las 75 temporadas de la historia. Faltó una sola vez: en 1980, cuando el GP de Italia se corrió en Imola. |

## Cómo está armado

- **Modelo de datos:** 14 tablas relacionadas exclusivamente por ID (nunca por texto/nombre), en esquema de estrella.
- **DAX:** medidas validadas una por una contra registros reales de la F1 (récords de puntos, campeones, victorias por circuito).
- **Fuente de datos:** dataset histórico de resultados de F1 (Ergast/Kaggle), 1950-2024.
- **Herramienta:** Power BI Desktop.

## Lo que me llevo de este proyecto

Ninguno de estos hallazgos lo busqué de antemano. Todos aparecieron por sostener la misma regla en cada página: no dar un número por bueno hasta confirmarlo contra la historia real, y cuando algo no cerraba, buscar la causa exacta antes de tocar nada. Esa misma disciplina fue la que en su momento me destapó a Adolf Brudes con 1116 victorias imposibles, y meses después la que me hizo entender que Rindt no había corrido las últimas carreras de 1970 porque había muerto en Monza, o que Abu Dabi 2014 no era un error mío sino la única vez que la F1 jugó con puntos dobles.
