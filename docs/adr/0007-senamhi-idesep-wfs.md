# ADR-007: Integración WFS IDESEP SENAMHI + TIFFs como fuente de alertas y mapa en vivo

**Estado:** Aceptada · **Fecha:** 2026-06-26

## Contexto

El brief original proponía scraping del portal de avisos SENAMHI (`senamhi.gob.pe/?p=avisos`)
para extraer alertas activas, y Open-Meteo Archive para el gráfico comparativo histórico.

Durante la investigación previa al evento, el equipo identificó que el SENAMHI publica una
infraestructura de datos geoespaciales completa a través del **IDESEP** (Infraestructura de
Datos Espaciales del Perú) usando el estándar OGC **WFS** (Web Feature Service) y archivos
raster **TIFF** de precipitación.

Estos endpoints son APIs públicas estándar — no requieren scraping, no tienen protección CSRF,
y devuelven GeoJSON estructurado con filtros espaciales (BBOX por coordenada GPS).

## Decisión

Reemplazar el scraping del portal de avisos SENAMHI por **consultas directas al WFS de IDESEP**
y usar los **TIFFs** como overlay raster en el mapa Leaflet.

### Capas adoptadas

| Capa IDESEP | Formato | Datos | Uso en YakuAlert |
|---|---|---|---|
| `g_prono_pp_24h` | WFS GeoJSON | Polígonos de alerta con nivel y fecha | Alerta oficial → señal de vulnerabilidad (20%) |
| `g_03_05` | WFS GeoJSON | Predicción numérica 24/48/72h (prop. `prec`) | Señal de lluvia futura → refuerza score |
| `g_04_02` | WFS GeoJSON | Anomalía precipitación por estación (`codigo`) | Puntos en mapa con color por anomalía |
| `g_02_04` | WFS GeoJSON | Polígonos El Niño 2017 (DEF/EFM/FMA) | Gráfico comparativo histórico |
| `g_02_01` / `g_02_02` | WFS GeoJSON | Polígonos Niño 82-83 y 97-98 | Gráfico comparativo histórico |
| TIFFs 24/48/72h | Raster | Precipitación continua sobre todo el Perú | Overlay de calor en mapa Leaflet |

### Formato de consulta WFS

```
GET https://idesep.senamhi.gob.pe/geoserver/{capa}/wfs
    ?service=WFS&version=2.0.0&request=GetFeature
    &typeName={typename}&outputFormat=application/json
    &CQL_FILTER=BBOX(geom,{lon_min},{lat_min},{lon_max},{lat_max})
```

Buffer de 0.5° alrededor del GPS del usuario. Timeout de 3s con fallback a `danger-zones.json`.

### Mapa multicapa (orden de renderizado)

```
Capa 4 (top): Polígonos rojos — aviso lluvia intensa 24h
Capa 3:       Puntos estaciones — anomalía precipitación coloreada
Capa 2:       Círculos — cauces históricos (danger-zones.json)
Capa 1:       TIFF overlay — precipitación continua 24h
Capa 0:       OpenStreetMap base
```

Los TIFFs se renderizan con `georaster-layer-for-leaflet` (CDN, sin API key).

## Consecuencias

**Positivas:**
- Cero scraping de HTML frágil para las alertas SENAMHI.
- Datos geoespaciales en GeoJSON estándar — directamente pintables en Leaflet.
- Las alertas son oficiales y verificables — refuerza la credibilidad ante el jurado.
- Los históricos del Niño disponibles como WFS para el gráfico comparativo, sin depender de Open-Meteo Archive para ese dato.
- El mapa en vivo con TIFF + WFS es visualmente diferenciador frente a otros equipos.
- La llave `codigo` permite cruzar anomalía PP + Tmin + Tmax por estación en un join limpio.

**Negativas / Riesgos:**
- Los servidores IDESEP del Estado peruano pueden tener latencia variable.
- Se mitiga con timeout de 3s y fallback automático a datos estáticos.

## Alternativas descartadas

- **Scraping portal de avisos SENAMHI** — HTML frágil, sin estructura GeoJSON, sin filtro espacial.
- **Solo Open-Meteo Archive** para históricos — menos preciso que los polígonos oficiales del IDESEP que ya tienen el rango clasificado por evento.
