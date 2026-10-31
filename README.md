# ☕ Centinela del Café · Proyecto Centinela · Fase 1

**Autor:** Julio César González R. (individual) · Profundización I: Redes Neuronales — Deep Learning · Maestría en Ciencia de Datos, USTA · Propuesta de Datos v1 · 2026-10-30

## Propuesta en una mirada

| | |
| --- | --- |
| **Evento de riesgo** | Hoja de café robusta **enferma: sí/no** (roya o ácaro rojo). Las 6 clases originales se agrupan como sana (0) frente a cualquier afectación (1). |
| **Cliente** | Técnico de campo de un lote cafetero (CIIDEA-ESPAM, Calceta, Manabí, Ecuador; 390 plantas). La alerta le indica qué plantas inspeccionar primero. |
| **Imagen** | [RoCoLe v2](https://doi.org/10.17632/c5yvn32dzg.2), Mendeley Data · 1.560 fotos de hojas con polígono de la hoja · CC BY 4.0 |
| **Serie temporal** | [NASA POWER](https://power.larc.nasa.gov/), clima mensual en Calceta (temperatura, humedad relativa, precipitación) · datos NASA de libre uso |
| **Fase 1** | Se aplana la **imagen**: 14 rasgos de color y textura medidos dentro de la hoja → MLP en PyTorch. Partición **por planta** (60/20/20, semilla 42). |
| **Fase 2** | CNN sobre la foto + LSTM/GRU sobre el clima de los meses previos, unidas por fusión. |
| **Clave de alineación** | **Agregada:** (región = celda NASA POWER de Calceta, mes = 2019-04). RoCoLe no trae fecha ni GPS por foto, así que todas las hojas comparten el mismo clima (rasgo constante: no entra al MLP en la Fase 1). |
| **Línea base** | Clase mayoritaria y el escalar crudo `frac_roya` (fracción de píxeles color roya), con umbral elegido en validación. |
| **Por qué no es un reto resuelto** | Fotos crudas en 3 resoluciones, con fondo variable; no existe una tabla de rasgos. Los datos se descargan del productor primario con su huella SHA-256 verificada, no de réplicas. |
| **Datos personales** | Ninguno en lo que se modela. El CSV de anotaciones trae el correo del anotador: la columna se elimina al cargar (Ley 1581 de 2012). |

📄 **Propuesta completa** (secciones 0–8, prueba de acoplamiento de diez filas, test de procedencia y *checklist* de la Ley 1581): [`docs/propuesta_datos.md`](docs/propuesta_datos.md)

## Contenido del repositorio

| Ruta | Qué es |
| --- | --- |
| `Centinela_Fase1_Cafe.ipynb` | Cuaderno de la Fase 1 (se abre en Colab y corre con «Ejecutar todo») |
| `docs/propuesta_datos.md` | Propuesta de Datos completa con sus anexos A, B y C |

## Datos

Este repositorio **no contiene datos**. El cuaderno los descarga de sus fuentes primarias (celda 1) y comprueba la huella SHA-256 que publica Mendeley:

| Archivo | Fuente | SHA-256 |
| --- | --- | --- |
| `RoCoLE-csv.csv` | Mendeley Data, DOI 10.17632/c5yvn32dzg.2 | `a9aa6b41…5ac5205b` |
| `RoCoLe-voc.tar.gz` | Mendeley Data, DOI 10.17632/c5yvn32dzg.2 | `7b45c70c…16b485` |
| Clima mensual 2017–2019 | API de NASA POWER | `02da8ae0…62ebf235` (bloque de datos) |

## Referencias

- Parraga-Alava, J., Cusme, K., Loor, A. y Santander, E. (2019). *RoCoLe: A robusta coffee leaf images dataset* (Versión 2) [Conjunto de datos]. Mendeley Data. https://doi.org/10.17632/c5yvn32dzg.2
- National Aeronautics and Space Administration. (s. f.). *POWER: Prediction Of Worldwide Energy Resources* [Conjunto de datos]. https://power.larc.nasa.gov/

Las imágenes de RoCoLe se usan bajo CC BY 4.0; los rasgos tabulares de este proyecto son una obra derivada de RoCoLe.
