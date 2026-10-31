# Propuesta de Datos · Centinela del Café · Fase 1

**Autor:** Julio César González R. · **Fecha:** 2026-10-30 · **Versión:** v1

## 0. Encabezado

| Campo | Detalle |
| --- | --- |
| Título del proyecto | Centinela del Café: alerta temprana de roya y ácaro en hoja de café robusta |
| Integrantes | Julio César González R (individual) |
| Fecha | 2026-10-30 |
| Versión | v1 (Fase 1) |

## 1. Problema, cliente y objetivo binario

- **Evento de riesgo y cliente:** detectar, a partir de una foto de la hoja, si una planta de café robusta está enferma, para el técnico de campo de un lote cafetero como el del CIIDEA-ESPAM (Calceta, Manabí, Ecuador).
- **Impacto:** la roya (*Hemileia vastatrix*) defolia la planta y reduce la cosecha; si se detecta tarde, se extiende a las plantas vecinas.
- **Objetivo binario:** «hoja enferma: sí/no». RoCoLe trae 6 clases ordinales (sana, ácaro rojo, roya 1 a 4). Se agrupan como **sana (0)** frente a **cualquier afectación (1)**: ácaro o roya de cualquier nivel. Justificación: para el cliente toda hoja afectada exige inspección; separar severidades no cambia la acción inmediata. Se pierde la severidad, que se conserva para el análisis de errores por subgrupo.

## 2. Las dos modalidades del sistema completo

- **Rama de imagen:** 1.560 fotos RGB de hojas de café robusta (RoCoLe), tomadas con la cámara de un teléfono (5 MP) en el lote de 390 plantas del CIIDEA-ESPAM. Aportan el **síntoma visible**: manchas amarillo-naranja de roya, punteado de ácaro, pérdida de verde.
- **Rama temporal:** clima mensual de NASA POWER (temperatura media, máxima y mínima a 2 m; humedad relativa; precipitación corregida) en la celda de Calceta, de reanálisis satelital MERRA-2. Aporta las **condiciones que favorecen la roya** (humedad alta y lluvias prolongadas).
- **Pregunta de control:** sí. La foto muestra **dónde** y **cuánto** daño tiene cada hoja; el clima es el mismo para todas las hojas del lote y no puede saber cuál está enferma. Son dos instrumentos distintos (cámara frente a radiómetro/reanálisis) y ninguna se deriva de la otra.

## 3. Fuentes (con enlaces y licencia)

| Modalidad | Fuente (productor primario) | Enlace o DOI | Licencia y versión | Dónde se verificó la licencia | ¿Datos de personas? |
| --- | --- | --- | --- | --- | --- |
| Imagen | RoCoLe v2 (Parraga-Alava et al., Mendeley Data) | [doi.org/10.17632/c5yvn32dzg.2](https://doi.org/10.17632/c5yvn32dzg.2) | CC BY 4.0 | Ficha del conjunto en la API pública de Mendeley (`data_licence`: CC BY 4.0), consultada el 2026-10-30 | No en las fotos (hojas). El CSV trae el correo del anotador: se elimina al cargar (anexo C) |
| Temporal | NASA POWER, API mensual v2.10 | [power.larc.nasa.gov](https://power.larc.nasa.gov/) | Datos NASA de libre uso, sin registro ni token | Página de la fuente | No |

## 4. Por qué NO es un reto resuelto

- **Dato crudo:** las fotos llegan con fondo variable (suelo, otras hojas), luz irregular y **tres resoluciones** distintas (1280×720, 2048×1152 y 4128×2322). No hay tabla de rasgos: hay que construirla con el polígono de la hoja que trae la anotación. El CSV de Labelbox trae 18 columnas, 4 de ellas vacías en 99–100 % de las filas, un dato personal y un nombre de archivo irregular. La etiqueta original es ordinal y desbalanceada (roya 4: 30 hojas de 1.560).
- **Sin Kaggle:** los datos se descargan de Mendeley Data (productor primario), con la huella SHA-256 que publica el repositorio, recalculada en el cuaderno. Existen réplicas en Kaggle (anexo B); no se usan.
- **Sin solución que copiar:** los trabajos publicados sobre RoCoLe clasifican la imagen con CNN. Aquí el reto es otro: **aplanar** la imagen a rasgos de color y textura **dentro de la hoja**, partir **por planta** para evitar fuga, elegir el umbral por el **costo** de cada error y preparar la **fusión con el clima** de la Fase 2, que ningún trabajo público hace con estas fuentes.

## 5. Plan de la Fase 1, clave de alineación y prueba de acoplamiento

- **Modalidad que se aplana en la Fase 1:** la **imagen**. Cada foto se resume en 14 rasgos tabulares calculados dentro del polígono de la hoja (medias y dispersión de R, G, B; matiz, saturación y brillo; exceso de verde; fracción de píxeles color roya; fracción de píxeles oscuros; textura; área de la hoja). Modelo: MLP en PyTorch.
- **Paso a la Fase 2:** los rasgos se reemplazan por una **CNN** sobre la foto (aprendizaje por transferencia) y se incorpora la **serie climática** con una LSTM/GRU sobre los meses previos, unidas por fusión.
- **Clave de alineación: agregada**, **(región = celda NASA POWER de Calceta, −0,8342; −80,1772; mes = 2019-04)**. RoCoLe no trae fecha ni GPS por foto (`date_captured` es nulo en el archivo COCO). El lote está documentado en el artículo de los autores, y la plataforma de anotación registra la carga de las fotos entre el 14 y el 17 de mayo de 2019. Se toma abril de 2019, último mes completo antes de la carga.
- **Salvedades:** (1) **rasgo casi constante**: todas las hojas heredan el mismo vector climático, así que en la Fase 1 el clima no discrimina y **no entra al MLP**; (2) **fuga**: si en la Fase 2 se usan varias ventanas, la partición no mezclará la misma ventana entre entrenamiento y prueba; (3) el mes es una **aproximación**: la toma pudo ocurrir entre febrero y mayo de 2019 (temporada de lluvias de Manabí).
- **Prueba de acoplamiento:** diez filas reales en el anexo A, producidas por la celda 11 del cuaderno.

## 6. Línea base comprometida

La partición es **por planta** (60/20/20, semilla 42): las ≈ 4 hojas de una planta caen en un solo conjunto. Se reporta en prueba:

| Referencia | Qué se reporta |
| --- | --- |
| Clase mayoritaria del entrenamiento | Exactitud y *recall* de «enferma» (vale 0 si la mayoritaria es «sana») |
| ¿La modalidad aplanada resume un escalar? | No: es un vector de 14 rasgos. Aun así se reporta el escalar más cercano a la pregunta del cliente |
| Escalar crudo: `frac_roya` (fracción de píxeles color roya) | Umbral elegido en validación (máximo F1); exactitud, precisión, *recall* y F1 en prueba |

La partición, la semilla y estas cifras se conservan como punto de comparación de la Fase 2.

## 7. Riesgos éticos y protección de datos

| Riesgo | Daño para el cliente | Mitigación |
| --- | --- | --- |
| Sesgo de muestreo o representatividad | Un solo lote, una variedad (robusta), un teléfono y una temporada: el modelo puede fallar en café arábica, otra altitud u otra cámara | Se declara el alcance en el informe; partición por planta; en la Fase 2 se valida con otra fuente antes de recomendar uso en campo |
| Falsa alarma | Visita de inspección innecesaria; si se fumiga sin revisar, gasto en fungicida y daño ambiental | Política en dos pasos: la alerta ordena **inspeccionar**, no fumigar; se reporta la tasa de falsas alarmas |
| Alarma omitida | La roya esporula y se extiende a plantas vecinas; pérdida de cosecha | Umbral elegido en validación para un *recall* ≥ 0,90; análisis de escapes por severidad |
| Dato personal en la fuente | El CSV trae el correo de quien anotó | Se elimina al cargar; nunca se muestra ni se sube a un asistente de IA |

El *checklist* de protección de datos va en el anexo C, uno por fuente.

## 8. Cita APA 7 de cada fuente

- Parraga-Alava, J., Cusme, K., Loor, A. y Santander, E. (2019). *RoCoLe: A robusta coffee leaf images dataset* (Versión 2) \[Conjunto de datos\]. Mendeley Data. https://doi.org/10.17632/c5yvn32dzg.2
- Parraga-Alava, J., Cusme, K., Loor, A. y Santander, E. (2019). RoCoLe: A robusta coffee leaf images dataset for evaluation of machine learning based methods in plant diseases recognition. *Data in Brief, 25*, 104414. https://doi.org/10.1016/j.dib.2019.104414
- National Aeronautics and Space Administration. (s. f.). *POWER: Prediction Of Worldwide Energy Resources* \[Conjunto de datos\]. https://power.larc.nasa.gov/

## Anexo A · Prueba de acoplamiento (diez filas reales)

Clave: (región = celda NASA POWER de Calceta, mes = 2019-04). Clima de abril de 2019: T2M = 25,56 °C · RH2M = 85,65 % · PRECTOTCORR = 7,44 mm/día.

| N.º | Clave (región, mes) | Rama de imagen: archivo (clase original) | Serie: T2M · RH2M · PRECTOTCORR |
| --- | --- | --- | --- |
| 1 | (Calceta, 2019-04) | C1P1H1.jpg (healthy) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 2 | (Calceta, 2019-04) | C1P2H1.jpg (healthy) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 3 | (Calceta, 2019-04) | C1P3E1.jpg (healthy) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 4 | (Calceta, 2019-04) | C1P3H1.jpg (healthy) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 5 | (Calceta, 2019-04) | C1P33E1.jpg (red\_spider\_mite) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 6 | (Calceta, 2019-04) | C2P8E2.jpg (red\_spider\_mite) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 7 | (Calceta, 2019-04) | C1P18H2.jpg (rust\_level\_1) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 8 | (Calceta, 2019-04) | C1P2E2.jpg (rust\_level\_2) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 9 | (Calceta, 2019-04) | C1P3H2.jpg (rust\_level\_3) | 25,56 °C · 85,65 % · 7,44 mm/día |
| 10 | (Calceta, 2019-04) | C2P4E2.jpg (rust\_level\_4) | 25,56 °C · 85,65 % · 7,44 mm/día |

Código: celda 11 del cuaderno `Centinela_Fase1_Cafe.ipynb` (une `RoCoLE-csv.csv` con la respuesta de la API mensual de NASA POWER por `region` y `mes`; añade además el rasgo `frac_roya` de cada foto). Que las diez filas compartan el mismo clima **es** la salvedad de la sección 5: la alineación es por lote y periodo, no por hoja.

## Anexo B · Evidencias del test de procedencia

**1. Cadena de procedencia**

| Archivo (nombre exacto) | URL o DOI del productor primario | Fecha de consulta | Huella SHA-256 |
| --- | --- | --- | --- |
| RoCoLE-csv.csv (2.220.101 bytes) | Mendeley Data, DOI 10.17632/c5yvn32dzg.2 (carpeta Annotations) | 2026-10-30 | a9aa6b411569dba7e9ba56fed79e5cd82af04a0ef092e56309158d632ac5205b |
| RoCoLe-voc.tar.gz (697.591.506 bytes) | Mendeley Data, DOI 10.17632/c5yvn32dzg.2 (carpeta Annotations) | 2026-10-30 | 7b45c70c5a1e4270ff456ab95d07dca2c6533d15954015ba0261cdcdc016b485 |
| nasa\_power\_calceta\_2017\_2019.json | power.larc.nasa.gov, API mensual (T2M, T2M\_MAX, T2M\_MIN, RH2M, PRECTOTCORR; 2017–2019) | 2026-10-30 | 02da8ae0bf87836b17d02cd30c41a7b696f3815a4494dbf77ed4e55662ebf235 (bloque de datos) |

Las huellas de Mendeley son las que publica su API pública; la celda 1 del cuaderno las recalcula sobre el archivo descargado y se detiene si no coinciden. La respuesta de NASA POWER incluye tiempos de proceso que cambian en cada consulta; por eso la huella se calcula sobre `properties.parameter` serializado con claves ordenadas.

**2. Descarga trazable:** celda 1 del cuaderno. Obtiene la URL de descarga y la huella desde `data.mendeley.com/public-api/datasets/c5yvn32dzg` y el clima desde la API de NASA POWER. La búsqueda de la palabra prohibida en el cuaderno no devuelve coincidencias (0).

**3. Búsqueda declarada de espejos (2026-10-30):** tres coincidencias en Kaggle, no usadas: [nirmalsankalana/rocole-a-robusta-coffee-leaf-images-dataset](https://www.kaggle.com/datasets/nirmalsankalana/rocole-a-robusta-coffee-leaf-images-dataset), [diegopgonzlez/rocoleoriginal](https://www.kaggle.com/datasets/diegopgonzlez/rocoleoriginal/activity) y [jumenta/rocole-test](https://www.kaggle.com/datasets/jumenta/rocole-test/tasks).

**4. Huella de crudeza (antes de limpiar):** CSV de 1.560 filas × 18 columnas; 22,2 % de celdas vacías (Benchmark Agreement, Benchmark ID y Benchmark Reference ID al 100 %; Agreement al 99 %); un nombre irregular (`C11P11HE.jpg`); fotos en tres resoluciones (479 de 1280×720, 768 de 2048×1152 y 313 de 4128×2322). Etiqueta: 791 sanas (50,7 %) y 769 afectadas (49,3 %), con la severidad desbalanceada: ácaro 167, roya 1: 344, roya 2: 166, roya 3: 62, roya 4: 30.

**5. Contraste con el espejo:** las réplicas de Kaggle redistribuyen las fotos. Los archivos de este proyecto son los del productor primario, verificados contra la huella que este publica. Incluyen el CSV de Labelbox con el polígono de cada hoja, que el proyecto necesita para enmascarar el fondo.

## Anexo C · Checklist de protección de datos (Ley 1581 de 2012)

**Fuente 1: RoCoLe v2 (Mendeley Data).** Resultado: **sin datos de personas** en el conjunto que se modela, tras eliminar una columna con un dato personal del anotador.

- [x] A. Se revisaron columnas, imágenes y metadatos. Las fotos son hojas, sin rostros ni voces. El CSV trae `Created By`, el **correo de quien anotó** (identificador directo de un tercero): se elimina al cargar, antes de mostrar cualquier fila. Las fotos no traen GPS por imagen; la ubicación publicada es la de una finca institucional (CIIDEA-ESPAM), no de una persona.
- [x] B. Sin datos sensibles (salud, biométricos, origen étnico, etc.).
- [x] C. Sin datos de niños, niñas o adolescentes.
- [x] D. Fuente pública de un repositorio académico con licencia explícita; se citan sus términos (CC BY 4.0).
- [x] E. Finalidad: uso académico para entrenar y evaluar un modelo en el curso; no se toman decisiones sobre personas identificadas.
- [x] F. Se eliminaron el correo del anotador, la columna `Masks` (contiene un token de acceso de Labelbox) y las URL internas. Prueba de reidentificación: las 3 columnas conservadas (identificador, nombre de archivo y etiqueta con el polígono) no permiten señalar a ninguna persona.
- [x] G. CC BY 4.0, verificada en la ficha de Mendeley. Obligación BY: atribuir e indicar los cambios («rasgos tabulares derivados de RoCoLe»).
- [x] H. A Colab, Drive y al asistente de IA solo llegan datos sin personas identificables; al asistente no se le pegó el CSV con la columna personal.
- [x] I. Se conservan solo `ID`, `External ID` y `Label`. Los datos crudos se guardan en el Drive del estudiante y se borran al terminar el curso.
- [x] J. Repositorio privado compartido con el docente, o público sin los datos y con el script de descarga. Las salidas del cuaderno no muestran la columna eliminada.

**Fuente 2: NASA POWER.** Resultado: **sin datos de personas**. Variables meteorológicas en una celda de reanálisis; finalidad académica; datos NASA de libre uso, sin registro; se guardan en el Drive del estudiante y se borran al terminar el curso. Bloques B, C, D (datos propios), F y J no aplican.

**Declaración.** Lo consignado en este *checklist* es verdadero y se revisó para cada fuente de la Propuesta.

| Integrante | Firma | Fecha |
| --- | --- | --- |
| Julio César González |  | 2026-10-30 |
