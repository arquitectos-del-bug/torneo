# ADR-0004: Supabase (PostgreSQL) como persistencia

**Estado:** Aceptada · **Fecha:** 2026-06-26

## Contexto
Necesitamos persistir datos compartidos entre usuarios: alertas oficiales activas, caché de
caudales del COES, reportes comunitarios y logs de consultas. El perfil individual va en
LocalStorage ([ADR-0001](0001-sin-auth.md)), pero lo compartido requiere una base de datos.

## Decisión
Usar **Supabase** (PostgreSQL gestionado). Se configura en minutos, tiene plan gratuito,
SDK de JS/TS oficial y Row Level Security para exponer lectura pública de forma segura.

## Consecuencias
- ✅ Setup rápido; SQL estándar (PostgreSQL) conocido por el equipo.
- ✅ Lectura directa desde el cliente con la *anon key* + RLS; escritura solo con *service role*.
- ✅ Soporta datos geográficos si más adelante usamos PostGIS.
- ⚠️ Dependencia de un servicio externo → se mitiga con *mock fallback* en la demo
  (ver [deployment.md](../deployment.md)).
- ⚠️ Los datos estáticos (cauces peligrosos) **no** van aquí: se versionan en `data/danger-zones.json`.
