# Codigo-Tarea2 — Reducción, astrometría y detección de NGC 7793

Repositorio: <https://github.com/M4rt03/Codigo-Tarea2>

Notebook de la Tarea 2 de Instrumentación Astronómica: reducción de imágenes CCD de la galaxia **NGC 7793** en los filtros **B** y **V**, calibración astrométrica con Gaia DR3, calibración fotométrica con estrellas estándar, detección de fuentes y construcción de catálogos.

## Contenido

| Archivo | Descripción |
|---|---|
| `REDUCCION_DETECCION_FINAL.ipynb` | Notebook con todo el pipeline (reducción → astrometría → fotometría → detección → catálogos → imagen a color). |
| `wcs_15oct.fits` | Solución WCS de astrometry.net (sobre la grilla de V 723), usada como punto de partida de la astrometría. |
| `gaia_ngc7793.ecsv` | Estrellas de Gaia DR3 del campo (G < 17) descargadas de VizieR; el notebook la genera si no existe. |

## Pipeline

1. **Reducción**
   - Master dark: mediana de 5 darks de 120 s (mismo tiempo que la ciencia).
   - Master flats B y V: mediana de 10 flats por filtro, menos el nivel cero, normalizados.
   - Máscara de píxeles malos (`BPM.fit`) y esquinas sin luz (flat < `FLAT_MIN`) quedan en `NaN`.
   - Exposiciones usadas: B 727, 729, 731 y V 723, 725 (las demás se descartaron).
2. **Alineación y apilado**: detección de estrellas con `DAOStarFinder`, emparejamiento por votación del desplazamiento, transformación con `astroalign` y combinación por mediana.
3. **Astrometría**: WCS inicial de astrometry.net y ajuste final (proyección TAN) contra Gaia DR3 corregida por movimiento propio, con `fit_wcs_from_points`. Se escriben `Img_B.fits` e `Img_V.fits`.
4. **Estrellas estándar** (HD 220881 y HD 8130, 26/10): reducción propia (nivel cero con darks de 5 s, corriente oscura escalada), fotometría de apertura, zeropoints, extinción y término de color de B.
5. **Detección de fuentes** con `photutils` (resta de fondo, filtrado con el seeing, segmentación y *deblending*) en dos configuraciones:
   - **agresiva**: separa estrellas y nudos de la galaxia;
   - **extendida**: mantiene la galaxia y las estructuras grandes como un solo objeto.
6. **Magnitudes**: `m = ZP − k·X − 2.5·log10(flujo)`, con corrección por término de color en B.
7. **Catálogos** con RA/Dec (ICRS) en `.ecsv` y `.csv`: `catalogo_V_agresiva`, `catalogo_B_agresiva`, `catalogo_BV`, `catalogo_V_extendida`, `catalogo_B_extendida`.
8. **Imagen a color** (R = V, G = (B+V)/2, B = B) con el estiramiento asinh de Lupton et al. (2004).

## Requisitos

Python 3 con:

```bash
pip install numpy scipy matplotlib astropy photutils astroalign jupyter
```

## Uso

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/M4rt03/Codigo-Tarea2.git
   cd Codigo-Tarea2
   ```
2. Copiar en la misma carpeta los datos crudos (no incluidos por su tamaño):
   - ciencia: `NGC_7793_B_120.000secs_000007{27..31}.fit`, `NGC_7793_V_120.000secs_000007{23..26}.fit`
   - darks: `Dark_120.000secs_00000{774..778}.fit`
   - flats: `FLAT_B_B11-{001..010}.fit`, `FLAT_V_B11-{001..010}.fit`
   - máscara: `BPM.fit`
   - estrellas estándar: archivos `HIP_115796*` y `HIP_6257*` de `Estrellas_estandar_data` (o ajustar `DIR_STD`)
3. Ajustar la variable `SALIDA` de la primera celda de código a una carpeta propia (ahí se guardan figuras y catálogos).
4. Abrir y ejecutar el notebook:
   ```bash
   jupyter notebook REDUCCION_DETECCION_FINAL.ipynb
   ```

## Autor

Martín — [@M4rt03](https://github.com/M4rt03)
