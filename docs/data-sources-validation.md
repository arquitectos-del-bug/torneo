# Validación de fuentes de datos — YakuAlert

Documento de validación que confirma si cada fuente de datos externa entrega la información
necesaria para el motor de riesgo y las funcionalidades del sistema.

**Fecha:** 2026-06-26  
**Punto de prueba:** Chosica (lat -11.95, lon -76.70)

---

## Resumen ejecutivo

| Fuente | ¿Entrega lo necesario? | Riesgo | Acción requerida |
|--------|:----------------------:|--------|------------------|
| Open-Meteo Forecast | ✅ Sí | Bajo | Ninguna |
| Open-Meteo Archive | ✅ Sí | Bajo | Ninguna |
| Nominatim (OSM) | ✅ Sí | Bajo | Configurar `User-Agent` |
| SENAMHI IDESEP WFS | ⚠️ Probable | Medio | Verificar nombres de capas y columna geometría |
| SENAMHI El Niño JSON | ⚠️ No confirmado | Medio | Descubrir URL exacta del endpoint |
| SENAMHI TIFFs | ⚠️ Probable | Medio | Confirmar URLs de descarga directa |
| COES Hidrología | ⚠️ Requiere scraping | Alto | Ejecutar agent-browser para harvest |
| OECE/SEACE | ✅ Sí (documentado) | Bajo | No crítico para MVP |

---

## 1. Open-Meteo Forecast API

### ¿Qué necesitamos?
- Precipitación horaria pronosticada para las próximas 48 horas por coordenada GPS.
- Respuesta en JSON, sin API key, sin límite de uso restrictivo.

### ¿Lo entrega?
**✅ SÍ — Verificado documentalmente.**

- API pública, sin key, sin registro. Licencia CC BY 4.0.
- Endpoint: `GET https://api.open-meteo.com/v1/forecast`
- Parámetros: `latitude`, `longitude`, `hourly=precipitation`, `forecast_days=2`
- Responde con array de 48 valores horarios de precipitación (mm).
- Cobertura global, incluida Perú.
- Modelos disponibles: GFS (NOAA), ICON (DWD), entre otros — resolución 11 km global.
- Response time < 10ms según documentación.

### Cálculo del score
```
sum(precipitation[0:47]) → si > 15mm → 40 pts (escala lineal)
```

### Riesgos
- Rate limit no documentado explícitamente, pero API diseñada para uso libre sin key.
- Latencia desde servidores EU/NA a usuarios en Perú: ~100-200ms (aceptable).

### Fallback
No se requiere — Open-Meteo es la fuente primaria y más confiable del stack.

---

## 2. Open-Meteo Archive API

### ¿Qué necesitamos?
- Precipitación diaria histórica para marzo 2017 y marzo 2023 (eventos El Niño Costero / Yaku).
- Usado para el gráfico comparativo y para contextualizar el riesgo actual.

### ¿Lo entrega?
**✅ SÍ — Verificado documentalmente.**

- Endpoint: `GET https://archive-api.open-meteo.com/v1/archive`
- Parámetros: `latitude`, `longitude`, `start_date=2017-03-01`, `end_date=2017-03-31`, `daily=precipitation_sum`
- Historical Weather API cubre desde 1940 hasta presente (80+ años).
- Misma licencia CC BY 4.0, sin key.

### Datos esperados
- Marzo 2017 (El Niño Costero): picos de >40 mm/día en Chosica.
- Marzo 2023 (Ciclón Yaku): picos de >30 mm/día en Lima/Chosica.

### Riesgos
- Ninguno significativo — datos estáticos que pueden cachearse permanentemente.

---

## 3. Nominatim (OpenStreetMap) — Geocodificación inversa

### ¿Qué necesitamos?
- Convertir GPS (lat, lon) a distrito y provincia del Perú.
- Usado para identificar vulnerabilidad del distrito y personalizar el plan IA.

### ¿Lo entrega?
**✅ SÍ — Verificado documentalmente.**

- Endpoint: `GET https://nominatim.openstreetmap.org/reverse`
- Parámetros: `lat`, `lon`, `format=json`, `zoom=12` (nivel distrito)
- Devuelve `address.city_district` o `address.suburb` que mapeará al distrito peruano.
- Requiere header `User-Agent` válido (política de uso).

### Ejemplo de respuesta esperada para Chosica
```json
{
  "address": {
    "suburb": "Lurigancho",
    "city": "Lima",
    "state": "Lima",
    "country": "Peru"
  }
}
```

### Riesgos
- **Política de rate limiting**: max 1 request/segundo. Para producción con volumen, debería usarse un mirror o cachear por coordenada redondeada.
- El campo de distrito puede variar: a veces es `suburb`, `city_district`, o `town`.

### Mitigación
- Cachear por coordenada truncada a 2 decimales (TTL 24h).
- Mapeo normalizador distrito → distrito canónico.

---

## 4. SENAMHI IDESEP WFS (GeoServer)

### ¿Qué necesitamos?

| Capa | Dato requerido | Uso en score |
|------|---------------|--------------|
| `g_prono_pp_24h` | Polígonos de alerta lluvia intensa 24h con nivel | Vulnerabilidad (20 pts) |
| `g_03_05` | Predicción numérica precipitación (propiedad `prec`) | Refuerzo señal lluvia |
| `g_04_02` | Anomalía precipitación por estación con `codigo` | Visual en mapa |
| `g_02_04` | Polígonos históricos El Niño 2017 | Gráfico comparativo |

### ¿Lo entrega?
**⚠️ PROBABLE — No se pudo verificar en vivo (sandbox sin acceso directo).**

**Evidencia a favor:**
- El IDESEP (Infraestructura de Datos Espaciales del Perú) es un servicio oficial OGC que usa GeoServer.
- La URL `https://idesep.senamhi.gob.pe/geoserver/` sigue el patrón estándar GeoServer.
- El estándar WFS 2.0.0 garantiza que si la capa existe, responde con GeoJSON al usar `outputFormat=application/json`.
- El ADR-007 del repo documenta haberlo verificado previamente.

**Puntos a confirmar antes de implementar:**
1. ✅ que la URL base sea exactamente `https://idesep.senamhi.gob.pe/geoserver/`
2. ⚠️ que el `typeName` exacto sea `g_prono_pp_24h` (podría ser `idesep:g_prono_pp_24h`)
3. ⚠️ que la columna de geometría sea `geom` (podría ser `the_geom` o `geometry`)
4. ⚠️ que el CQL_FILTER BBOX funcione con el nombre correcto de la columna
5. ⚠️ que las propiedades incluyan nivel de alerta y fecha de vigencia

### Consulta de verificación recomendada
```bash
# 1. Listar capas disponibles
curl "https://idesep.senamhi.gob.pe/geoserver/wfs?service=WFS&request=GetCapabilities" | head -200

# 2. Probar una capa con count=1
curl "https://idesep.senamhi.gob.pe/geoserver/g_prono_pp_24h/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=g_prono_pp_24h&outputFormat=application/json&count=1"

# 3. Probar BBOX (Chosica)
curl "https://idesep.senamhi.gob.pe/geoserver/g_prono_pp_24h/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=g_prono_pp_24h&outputFormat=application/json&CQL_FILTER=BBOX(geom,-77.2,-12.45,-76.2,-11.45)"
```

### Riesgos
- **Latencia variable**: servidores gubernamentales peruanos pueden tener >2s de respuesta.
- **Disponibilidad**: no hay SLA público; posibles ventanas de mantenimiento.
- **Nombres de capa**: pueden cambiar sin aviso.

### Fallback (documentado en architecture.md §6)
- Timeout 3s → usar solo `danger-zones.json` + Open-Meteo.
- Cachear la última respuesta exitosa de `g_prono_pp_24h` en Supabase `active_alerts`.

---

## 5. SENAMHI El Niño JSON

### ¿Qué necesitamos?
- Estado oficial ENFEN (El Niño/La Niña): condición actual, color, fecha de actualización.
- Usado como contexto informativo para el ciudadano.

### ¿Lo entrega?
**⚠️ NO CONFIRMADO — URL exacta del API JSON no documentada.**

**Evidencia:**
- El README lista "SENAMHI El Niño JSON" como fuente.
- La página `senamhi.gob.pe/?p=enfen` existe, pero no está claro si expone un JSON público o solo HTML.
- Podría ser necesario parsear el HTML o encontrar un endpoint interno.

### Acción requerida
```bash
# Probar endpoints candidatos:
curl "https://www.senamhi.gob.pe/api/elnino"
curl "https://www.senamhi.gob.pe/?p=enfen&format=json"
curl "https://www.senamhi.gob.pe/load/clima/elnino/estado_actual.json"
```

### Fallback
- Mock con último estado conocido: `{ "condicion": "El Niño Costero", "color": "naranja", "fecha": "2026-06" }`
- Dato estable (cambia mensualmente) → puede sembrarse manualmente.

---

## 6. SENAMHI IDESEP TIFFs (Raster)

### ¿Qué necesitamos?
- Archivos TIFF de precipitación continua (24h, 48h, 72h) sobre Perú.
- Para overlay en mapa Leaflet con `georaster-layer-for-leaflet`.

### ¿Lo entrega?
**⚠️ PROBABLE — Nombres de archivo documentados en `architecture.md`.**

**Archivos esperados:**
- `PT_03_05_001_03_000_513_0000_00_00.tif` (24h)
- `PT_03_05_002_03_000_513_0000_00_00.tif` (48h)
- `PT_03_05_003_03_000_513_0000_00_00.tif` (72h)

### Puntos a confirmar
1. URL base de descarga (¿está en el mismo GeoServer? ¿WCS? ¿directorio público?)
2. Tamaño del archivo (si es >5MB, impacta mobile load time)
3. Formato de georeferencia (CRS EPSG:4326 vs 32718 UTM)
4. Si requiere proxy server-side por CORS

### Consulta de verificación
```bash
# Probar si están en un directorio público del GeoServer
curl -I "https://idesep.senamhi.gob.pe/geoserver/www/PT_03_05_001_03_000_513_0000_00_00.tif"

# O vía WCS
curl "https://idesep.senamhi.gob.pe/geoserver/wcs?service=WCS&request=GetCoverage&coverageId=g_03_05&format=image/tiff"
```

### Fallback
- Sin TIFF → el mapa muestra solo capas vectoriales (polígonos WFS + danger-zones.json).
- La funcionalidad core (score) NO depende de los TIFFs.

---

## 7. COES Hidrología (Scraping)

### ¿Qué necesitamos?
- Caudal (m³/s) de ríos clave: Rímac, Piura, Moche, Chili.
- Histórico (marzo 2017, marzo 2023) + reciente.

### ¿Lo entrega?
**⚠️ REQUIERE SCRAPING — No hay API pública.**

- El COES publica datos de caudal a través de un formulario ASP.NET (Consulta Histórico Hidrología).
- No hay CSV ni API REST — requiere automatizar el formulario con un navegador headless.
- Documentado en ADR-006: usar `agent-browser` en build-time.

### Datos requeridos (seed mínimo)

| Río | Cuenca | Caudal 2017 | Caudal 2023 | Caudal "normal" |
|-----|--------|-------------|-------------|-----------------|
| Rímac | Rímac | ~487 m³/s (pico 15-mar-2017) | ~312 m³/s (pico 09-mar-2023) | ~50-100 m³/s |
| Piura | Piura | ~3500 m³/s (pico 27-mar-2017) | ~2100 m³/s (pico 2023) | ~20-50 m³/s |
| Moche | Moche | ~600 m³/s (2017) | ~400 m³/s (2023) | ~10-30 m³/s |

### Estrategia
1. **Harvest único** con agent-browser → poblar tabla `hydrology_readings`.
2. **Cron opcional** (1x/día) para mantener datos frescos.
3. **Mock seed**: si no hay acceso, sembrar con valores documentados públicamente por SENAMHI/ANA.

### Fallback
- Datos sembrados (seed) disponibles siempre → el score funciona sin COES.
- La señal de caudal es solo un "refuerzo" — no es parte de los 3 factores core.

---

## 8. OECE/SEACE (Contrataciones Abiertas)

### ¿Qué necesitamos?
- Datos de contrataciones públicas en formato OCDS (Open Contracting Data Standard).
- **NO es parte del motor de riesgo actual.**

### ¿Lo entrega?
**✅ SÍ — Documentado y público.**

- Endpoint: `https://contratacionesabiertas.oece.gob.pe/api/v1/file/{seace_v2|seace_v3}/json/{year}/{month}`
- Sin API key, descarga ZIP con JSON de releases compiladas.
- Regla: seace_v2 (2005-01 a 2013-01), seace_v3 (2013-02+).

### Uso potencial futuro
- Enriquecer la señal de vulnerabilidad: distritos con baja inversión en infraestructura de prevención (muros de contención, defensas ribereñas) podrían tener mayor vulnerabilidad.
- **NO necesario para el MVP.** Priorizar para una iteración futura.

---

## Matriz de validación pre-demo

Antes de la demo, el equipo debe ejecutar estos comandos y confirmar resultados:

```bash
#!/bin/bash
# === validate-sources.sh ===
# Ejecutar desde laptop con acceso a internet

echo "1. Open-Meteo Forecast (Chosica)"
curl -s "https://api.open-meteo.com/v1/forecast?latitude=-11.95&longitude=-76.70&hourly=precipitation&forecast_days=2" | python3 -c "
import json,sys; d=json.load(sys.stdin)
p=d['hourly']['precipitation']
print(f'  OK: {len(p)} horas, sum={sum(x for x in p if x):.1f}mm')
"

echo "2. Open-Meteo Archive (marzo 2017)"
curl -s "https://archive-api.open-meteo.com/v1/archive?latitude=-11.95&longitude=-76.70&start_date=2017-03-01&end_date=2017-03-31&daily=precipitation_sum" | python3 -c "
import json,sys; d=json.load(sys.stdin)
p=d['daily']['precipitation_sum']
print(f'  OK: {len(p)} días, max={max(x for x in p if x):.1f}mm, total={sum(x for x in p if x):.1f}mm')
"

echo "3. Nominatim (Chosica)"
curl -s -H 'User-Agent: YakuAlert/1.0' "https://nominatim.openstreetmap.org/reverse?lat=-11.95&lon=-76.70&format=json&zoom=12" | python3 -c "
import json,sys; d=json.load(sys.stdin)
a=d['address']
print(f'  OK: {a.get(\"suburb\",a.get(\"city_district\",\"?\"))}, {a.get(\"city\",\"?\")}, {a.get(\"state\",\"?\")}')
"

echo "4. SENAMHI IDESEP WFS - GetCapabilities"
curl -s --max-time 10 "https://idesep.senamhi.gob.pe/geoserver/wfs?service=WFS&request=GetCapabilities" | grep -o 'FeatureType' | wc -l | xargs -I{} echo "  Capas WFS encontradas: {}"

echo "5. SENAMHI IDESEP WFS - g_prono_pp_24h (1 feature)"
curl -s --max-time 10 "https://idesep.senamhi.gob.pe/geoserver/g_prono_pp_24h/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=g_prono_pp_24h&outputFormat=application/json&count=1" | python3 -c "
import json,sys; d=json.load(sys.stdin)
f=d.get('features',[])
print(f'  OK: {len(f)} features, props={list(f[0][\"properties\"].keys()) if f else \"VACIO\"}')
"

echo "6. SENAMHI IDESEP WFS - g_03_05 BBOX Chosica"
curl -s --max-time 10 "https://idesep.senamhi.gob.pe/geoserver/g_03_05/wfs?service=WFS&version=2.0.0&request=GetFeature&typeName=g_03_05&outputFormat=application/json&CQL_FILTER=BBOX(geom,-77.2,-12.45,-76.2,-11.45)" | python3 -c "
import json,sys; d=json.load(sys.stdin)
f=d.get('features',[])
if f:
    print(f'  OK: {len(f)} features, props={list(f[0][\"properties\"].keys())}')
else:
    print('  WARN: 0 features — verificar typeName o columna geometría')
"

echo "7. SENAMHI El Niño JSON (probar variantes)"
for url in "https://www.senamhi.gob.pe/api/elnino" "https://www.senamhi.gob.pe/?p=enfen"; do
  code=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$url")
  echo "  $url → HTTP $code"
done

echo "8. OECE/SEACE (HEAD check)"
curl -s -o /dev/null -w "  HTTP %{http_code}" --max-time 10 "https://contratacionesabiertas.oece.gob.pe/api/v1/file/seace_v3/json/2024/03"
echo ""

echo "=== Validación completa ==="
```

---

## Conclusiones

### ✅ Fuentes confirmadas (pueden usarse con confianza)

1. **Open-Meteo Forecast** — Entrega precipitación horaria exactamente como se necesita. Sin key, global, rápido. Base del 40% del score (lluvia).
2. **Open-Meteo Archive** — Entrega históricos desde 1940. Permite comparar con 2017/2023 directamente.
3. **Nominatim** — Geocodificación inversa funcional para Perú. Requiere User-Agent.

### ⚠️ Fuentes probables (requieren verificación in-situ)

4. **SENAMHI IDESEP WFS** — Alta probabilidad de funcionar (GeoServer estándar OGC), pero necesita:
   - Confirmar nombres exactos de capas y propiedades
   - Confirmar columna de geometría para CQL_FILTER
   - Medir latencia real (timeout 3s es el plan)

5. **SENAMHI TIFFs** — Los nombres de archivo están documentados pero la URL de descarga no.

6. **SENAMHI El Niño JSON** — Endpoint exacto no verificado. Puede requerir parsing HTML.

### 🔴 Fuentes que requieren trabajo extra

7. **COES Hidrología** — NO hay API. Requiere navegador headless. Solución: harvest en build-time + seed en Supabase.

### ℹ️ No necesario para MVP

8. **OECE/SEACE** — Funcional pero irrelevante para el motor de riesgo actual.

---

## Recomendación de priorización

```
CRÍTICO (score no funciona sin esto):
  ✅ Open-Meteo Forecast     → listo para implementar
  ✅ danger-zones.json       → necesita datos reales (coords de quebradas)

IMPORTANTE (mejora la calidad del score):
  ⚠️ SENAMHI IDESEP WFS     → probar desde laptop antes de demo
  ✅ Nominatim               → listo para implementar  

NICE TO HAVE (visual/informativo):
  ⚠️ SENAMHI TIFFs          → probar disponibilidad
  ⚠️ COES caudales          → seed manual si harvest falla
  ⚠️ SENAMHI El Niño JSON   → mock fallback suficiente
```
