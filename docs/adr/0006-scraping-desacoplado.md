# ADR-0006: Scraping COES desacoplado (agent-browser en build-time)

**Estado:** Aceptada · **Fecha:** 2026-06-26

## Contexto
El **COES** publica caudales históricos de ríos (Rímac, Mantaro, etc.) en su "Consulta Histórico
Hidrología" (pública, sin login), pero a través de un **formulario ASP.NET sin API/CSV**. El
caudal es una señal directa de desborde y la base de la comparativa 2017/2023.

Una opción es manejar el formulario con un navegador headless (`agent-browser` de vercel-labs,
que controla Chromium vía CDP). Pero correr un navegador headless **en runtime** sobre Vercel
serverless implica `@sparticuz/chromium`, cold starts lentos, límites de memoria y configuración
frágil — un punto único de falla justo en la demo, y contrario a [ADR-0002](0002-vercel-vs-sam.md).

## Decisión
**Desacoplar** el scraping del runtime. El navegador headless **nunca** atiende requests del
usuario. Se usa `agent-browser` como herramienta de **build-time**:
1. **Harvest único** de los caudales históricos 2017/2023 (no cambian) → seed en Supabase/JSON.
2. **Reverse-engineering** del POST del formulario COES → replicarlo con `fetch` plano.
3. *(Opcional)* **cron** desacoplado (GitHub Action / Vercel Cron) 1×/día → escribe en Supabase.

El runtime (`/api/hydrology`) **solo lee** de Supabase/seed.

## Consecuencias
- ✅ Runtime liviano y desplegable en segundos; sin navegador en el camino del usuario.
- ✅ Arquitectura defendible: separación clara entre ingestión de datos y servicio.
- ✅ Lo histórico se sirve aunque COES esté caído (dato sembrado); con *mock fallback* garantizado.
- ✅ Uso de IA/automatización para lo tedioso (descubrir el endpoint), avalado por las bases.
- ⚠️ El caudal "en vivo" puede tener hasta 24 h de desfase (cron diario) — aceptable para alerta temprana.
- ⚠️ Si COES cambia el formulario, hay que re-descubrir el endpoint (proceso aislado, no afecta runtime).
