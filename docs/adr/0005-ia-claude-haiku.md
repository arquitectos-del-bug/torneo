# ADR-0005: Claude Haiku 4.5 server-side para el plan de evacuación

**Estado:** Aceptada · **Fecha:** 2026-06-26

## Contexto
El plan de evacuación se genera con IA a partir del score, distrito y pronóstico. La idea inicial
proponía gpt-4o-mini con la API key **pegada por el usuario** en un panel de configuración. Eso
expone la key en el cliente y agrega fricción. Necesitamos un modelo rápido, barato y seguro.

## Decisión
Usar **Claude Haiku 4.5** (`claude-haiku-4-5`) vía `@anthropic-ai/sdk`, invocado **server-side**
desde `/api/recommendations`, con la API key en **variable de entorno** (`ANTHROPIC_API_KEY`).

## Consecuencias
- ✅ La key nunca viaja al cliente ni la tiene que pegar el usuario.
- ✅ Haiku es rápido y económico → ideal para generar 3 viñetas concisas con baja latencia.
- ✅ La IA solo **traduce** el resultado del motor a lenguaje accionable; no calcula el riesgo
  (coherente con [ADR-0003](0003-motor-heuristico.md)).
- ⚠️ Dependencia de un proveedor → si la IA falla, se muestran recomendaciones estáticas de respaldo.
- ⚠️ Costo por token (mínimo en Haiku); se acota con prompts breves y respuestas cortas.

## Alternativas consideradas
- **OpenAI gpt-4o-mini con key del usuario** — descartada por exponer la key y añadir fricción.
- **Sin IA (solo plantillas)** — descartada: la personalización por distrito aporta valor real.
