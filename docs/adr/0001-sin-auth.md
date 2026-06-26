# ADR-0001: Sin autenticación — perfil en LocalStorage

**Estado:** Aceptada · **Fecha:** 2026-06-26

## Contexto
YakuAlert es una app de **emergencia pública**. El usuario la abre en medio de una lluvia o un
posible desastre para saber si está en riesgo. Cada segundo y cada paso de fricción cuenta.

## Decisión
**No** implementar registro ni login (descartamos AWS Cognito y equivalentes). El perfil y la
configuración del usuario (ubicación guardada, último score, preferencias) se persisten en
**LocalStorage**, en su propio dispositivo.

## Consecuencias
- ✅ Cero fricción: el ciudadano ve su riesgo al instante, sin crear cuenta ni validar correo.
- ✅ Menos superficie de ataque y cero gestión de PII / contraseñas.
- ✅ Más tiempo de hackathon para el valor central (motor de riesgo, mapa, IA).
- ⚠️ El perfil no se sincroniza entre dispositivos (aceptable: el riesgo es local y efímero).
- ⚠️ Los reportes comunitarios serán anónimos; se mitigará con validación básica anti-spam.
