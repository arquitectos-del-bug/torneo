# 🌊 YakuAlert
> Alerta temprana de huaicos y desbordes que calcula tu riesgo en tiempo real según tu GPS,
> lo compara con los peores desastres históricos (2017 y 2023) y genera un plan de evacuación con IA.
> *Yaku* = "agua" en quechua.

---

## 🎯 Problemática que resuelve

Perú enfrenta huaicos y desbordes recurrentes (El Niño Costero 2017, Ciclón Yaku 2023) que
causan víctimas **por falta de aviso oportuno y accionable**. Ante un nuevo Fenómeno del Niño
en camino, las herramientas oficiales (SENAMHI, INDECI) publican boletines genéricos, técnicos
y a nivel de distrito; no responden la única pregunta que le importa al ciudadano:

> **¿Yo, donde estoy parado ahora mismo, estoy en riesgo y qué debo hacer?**

**Usuario objetivo:** cualquier ciudadano en zonas vulnerables del Perú (Lurigancho-Chosica,
Piura, valles costeros), especialmente quienes viven cerca de quebradas y cauces de ríos.
Acceso público, **sin registro**, optimizado para celular.

---

## 🎯 ¿Quién se beneficia, cuándo, cómo y por qué?

| | |
|---|---|
| **Who** | Ciudadanos en zonas de riesgo: Chosica, Catacaos, Trujillo, Arequipa. También autoridades locales de defensa civil. |
| **What** | Score de riesgo personalizado (0–100%) + mapa de peligros + plan de evacuación en lenguaje peruano. |
| **When** | Durante eventos climáticos activos y en las horas previas al desastre — la ventana crítica donde una alerta oportuna salva vidas. |
| **Where** | Cualquier zona del Perú con cobertura SENAMHI. Prioriza: Río Rímac (Chosica), Río Piura (Catacaos), Río Moche (Trujillo), Río Chili (Arequipa). |
| **Why** | Las herramientas oficiales están diseñadas para técnicos. Un ciudadano con señal baja no puede interpretar un PDF de 40 páginas en medio de una emergencia. |
| **How** | Motor de reglas explicable que cruza 3 señales verificables: cercanía a quebradas (Haversine) + lluvia 48h (Open-Meteo) + alertas oficiales (SENAMHI IDESEP WFS). La IA solo traduce el score a un plan concreto. |

---

## 💡 Qué hace

1. **Riesgo personalizado (0–100%)** calculado en vivo a partir de tu ubicación GPS.
2. **Mapa en vivo multicapa** — TIFFs de precipitación 24/48/72h como overlay + puntos de estaciones SENAMHI con anomalía de precipitación + polígonos de alerta oficial 24h.
3. **"Termómetro" del río:** compara el caudal actual con el de marzo 2017 y 2023 en tu cuenca, usando datos históricos del COES y capas WFS del IDESEP.
4. **Plan de acción con IA:** 3 recomendaciones concretas en lenguaje peruano (mochila de emergencia, rutas y zonas seguras de tu distrito).

El riesgo **no** lo inventa una IA opaca: es un **motor de reglas explicable** (ver
[`docs/architecture.md`](docs/architecture.md)) que combina tres señales reales y verificables.

---

## 🛠️ Stack tecnológico

| Capa | Tecnología | Por qué |
|------|------------|---------|
| Frontend | Next.js 14 (App Router) + TypeScript + Tailwind CSS | Front + API en un repo |
| Mapas | react-leaflet + georaster-layer-for-leaflet + OpenStreetMap | TIFFs raster + vectores sin API key |
| Backend | Next.js Route Handlers (serverless) | Deploy en `git push` |
| Persistencia | Supabase (PostgreSQL) + LocalStorage | Setup en minutos / perfil sin registro |
| IA | **Claude Haiku 4.5** (`claude-haiku-4-5`, server-side) | Rápido, barato, key en el servidor |
| Despliegue | Vercel | Sin AWS SAM/Cognito (ver ADRs) |

### APIs y fuentes de datos (verificadas, libres, sin tarjeta)

| Fuente | Formato | Uso |
|--------|---------|-----|
| Open-Meteo Forecast | JSON | Lluvia pronosticada 48h por lat/lon |
| Open-Meteo Archive | JSON | Acumulado histórico marzo 2017 / 2023 |
| Nominatim (OSM) | JSON | GPS → distrito/provincia |
| SENAMHI El Niño | JSON | Estado oficial ENFEN en vivo |
| SENAMHI IDESEP `g_04_02` | WFS GeoJSON | Anomalía precipitación actual por estación |
| SENAMHI IDESEP `g_prono_pp_24h` | WFS GeoJSON | Polígonos de aviso lluvia intensa 24h |
| SENAMHI IDESEP `g_03_05` | WFS GeoJSON | Predicción numérica precipitación 24/48/72h |
| SENAMHI IDESEP `g_02_04` | WFS GeoJSON | Histórico El Niño 2017 (comparativa) |
| SENAMHI IDESEP `g_02_01` / `g_02_02` | WFS GeoJSON | Histórico Niño 82-83 y 97-98 |
| SENAMHI IDESEP TIFFs | Raster | Precipitación continua 24/48/72h para overlay en mapa |
| COES Histórico Hidrología | Scraping desacoplado | Caudal (m³/s) del Rímac y cuencas clave |

---

## ▶️ Cómo correr el proyecto localmente

> ⚠️ **Sprint 0 (Hito 1):** este repositorio contiene únicamente el **prototipo de
> arquitectura y la documentación**. El código de la aplicación se implementa en el Sprint 1/2.

```bash
# 1. Clonar e instalar
git clone <repo-url> && cd yakualert
npm install

# 2. Configurar variables de entorno
cp .env.example .env.local   # completar ANTHROPIC_API_KEY y credenciales Supabase

# 3. Levantar en desarrollo
npm run dev                  # http://localhost:3000

# 4. Pruebas (motor de riesgo)
npm run test

# 5. Build de producción
npm run build
```

---

## 🤖 Modelos y herramientas de IA

| Herramienta | Rol |
|---|---|
| **Claude Haiku 4.5** (`claude-haiku-4-5`, server-side) | Genera el plan de evacuación personalizado. Prompt base en [`docs/prompts/evacuation-plan.md`](docs/prompts/evacuation-plan.md) |
| **agent-browser** (vercel-labs) | Herramienta de build-time para descubrir el endpoint POST del COES y cosechar caudales históricos. **No corre en producción** (ver [ADR-0006](docs/adr/ADR-006-scraping-desacoplado.md)) |
| **Asistentes IA (Claude, Copilot)** | Copiloto de desarrollo — cada decisión técnica documentada en los ADRs |

---

## 👥 Integrantes y roles

| Integrante | Rol |
|------------|-----|
| **Roberto Orellana Aliano** | Backend & Arquitectura — Route Handlers, motor de riesgo, Supabase, scraping COES |
| **Ricardo Manuel Solís Vizurraga** | Frontend & UX — UI Next.js, react-leaflet, capas raster TIFF, Tailwind, gráfico histórico |
| **David López Félix** | Datos, IA & DevOps — integración SENAMHI IDESEP WFS, prompts IA, deploy Vercel, tests |

---

## 📚 Documentación adicional

| Documento | Descripción |
|---|---|
| [`docs/architecture.md`](docs/architecture.md) | C4 (Contexto + Contenedores) + diagrama de secuencia + motor de riesgo |
| [`docs/data-model.md`](docs/data-model.md) | Tablas Supabase + diagrama ER + datos estáticos |
| [`docs/deployment.md`](docs/deployment.md) | Vercel + Supabase + APIs externas + resiliencia |
| [`docs/prompts/evacuation-plan.md`](docs/prompts/evacuation-plan.md) | Prompt system/user para el plan IA + parámetros |
| [`docs/adr/`](docs/adr/) | 7 decisiones de arquitectura justificadas |

---

*Proyecto del Torneo de VibeCoding — Developer Student Club PUCP, 26 de junio de 2026.*
