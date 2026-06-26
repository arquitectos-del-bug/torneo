# ADR-0003: Motor de riesgo heurístico (reglas) en vez de ML

**Estado:** Aceptada · **Fecha:** 2026-06-26

## Contexto
El corazón del producto es calcular un riesgo de huaico/desborde. Un modelo de ML entrenado
requeriría datos etiquetados, tiempo de entrenamiento e infraestructura — inviable en una
hackathon — y sería una **caja negra** difícil de sustentar ante el jurado.

## Decisión
Implementar un **motor de reglas heurísticas** como **función pura** (`lib/risk-engine.ts`) que
calcula un score 0–100 por suma ponderada de tres señales reales:
- Cercanía a un cauce peligroso (Haversine) — 40 pts.
- Lluvia pronosticada 48 h (Open-Meteo) — 40 pts.
- Vulnerabilidad del distrito (alerta COEN / histórico) — 20 pts.
- *(Opcional)* refuerzo por caudal actual del COES vs histórico.

## Consecuencias
- ✅ **Explicable:** podemos defender cada punto del score ante el jurado (rúbrica: Arquitectura 40%).
- ✅ **Testeable:** al ser función pura sin I/O, cubre el requisito de tests del Hito 2
  (happy path + caso de error) de forma trivial.
- ✅ Determinista y rápido, sin costo de inferencia.
- ⚠️ Los pesos son heurísticos, no aprendidos → se documentan como supuestos y son ajustables.
- ⚠️ No captura patrones complejos; aceptable para un sistema de alerta temprana orientado a la acción.
