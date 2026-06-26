# ADR-0002: Vercel + Next.js en vez de AWS SAM/Cognito

**Estado:** Aceptada · **Fecha:** 2026-06-26

## Contexto
Disponemos de ~6 horas de desarrollo. Configurar AWS SAM (CloudFormation, API Gateway, políticas
IAM) es potente pero riesgoso: un error de permisos en la consola de AWS puede bloquear al equipo
por horas. Necesitamos desplegar continuamente y sin sobresaltos.

## Decisión
Usar **Next.js (App Router) desplegado en Vercel**. Frontend (React) y backend (Route Handlers
serverless) viven en el **mismo repositorio** y se despliegan con `git push`.

## Consecuencias
- ✅ Deploy en segundos; el timestamp del deploy sirve como verificación del Hito 2.
- ✅ Un solo lenguaje (TypeScript) y un solo repo → menos contexto que cargar.
- ✅ Plan gratuito suficiente para la demo.
- ⚠️ Acoplamiento a Vercel; migrar a otra plataforma exigiría adaptar las funciones (riesgo bajo).
- ⚠️ Las funciones serverless tienen límites de tiempo/memoria → motiva [ADR-0006](0006-scraping-desacoplado.md)
  (no correr navegadores headless en runtime).
