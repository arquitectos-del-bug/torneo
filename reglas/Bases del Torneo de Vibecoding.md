# 

# **Bases del Torneo de VibeCoding**

## 

## Developer Student Club PUCP

#### 26 de junio de 2026

# 

---

1. ## **Resumen ejecutivo**

El Torneo de Vibecoding es una hackathon de un día organizada por el Developer Student Club PUCP donde equipos construyen software real, acelerado por inteligencia artificial generativa. El reto no es solo hacer que algo funcione, sino demostrar que hay ingeniería detrás de cada decisión.

**Objetivo general** Promover el uso responsable y técnico de herramientas de IA generativa en el desarrollo de software, fomentando la creación de prototipos funcionales con arquitectura sólida, trabajo en equipo y pensamiento crítico sobre el código producido.

**Fecha** Viernes 26 de junio del 2026

**Horario** 09:00 AM — 08:00 PM

**Ubicación** PUCP \- Pabellón V \- Aulas V201 y V202

**Formato** 10 equipos de 3 integrantes

**Link de inscripción: [https://gdg.community.dev/e/mrcs5u/](https://gdg.community.dev/e/mrcs5u/)**

**Brochure: [link](https://www.canva.com/design/DAHJOB4L-J4/PeUNV4ALBe70B-kF_uLwbw/view?utm_content=DAHJOB4L-J4&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h6c500f53a9)**

---

2. ## **Inscripción de participantes (la selección de equipos se hará según la motivación, conocimientos y experiencia de los integrantes)**

La participación es gratuita y por invitación. Los equipos se postulan y son seleccionados por los organizadores según los criterios descritos a continuación.

### **Requisitos para postular**

* Equipos conformados por exactamente 3 integrantes  
* Cada integrante debe adjuntar su CV actualizado, perfil de LinkedIn y perfil de GitHub  
* El equipo debe presentar una motivación grupal escrita explicando por qué quieren participar y qué esperan construir

### **Criterios de selección** 

La selección se realiza evaluando los conocimientos y experiencia técnica de los integrantes (proyectos previos, stack conocido, contribuciones en GitHub) y la calidad de la motivación grupal.

### **Postulación** 

A través del formulario oficial publicado en las fechas indicadas. El cierre de inscripciones es el 15 de junio del 2026\.

### **Anuncio de equipos seleccionados** 

20 de junio del 2026 vía correo electrónico. Los equipos seleccionados serán añadidos a un grupo de WhatsApp exclusivo donde recibirán información logística y recursos previos al evento

### **Uso de información proporcionada por los participantes**

Al postular al torneo, los participantes autorizan al equipo organizador a utilizar la información proporcionada durante el proceso de inscripción (incluyendo CV, perfil de LinkedIn, perfil de GitHub y demás información compartida) para fines de evaluación, organización del evento y actividades de reclutamiento vinculadas a empresas, startups y organizaciones aliadas.

La información podrá ser compartida con patrocinadores y aliados estratégicos interesados en identificar talento participante del evento, siempre en el marco de oportunidades profesionales, académicas o laborales relacionadas con los objetivos del torneo.

---

3. ## **Reto y temática**

   1. El reto Construir un producto de software funcional y desplegado en producción en menos de 11 horas, usando inteligencia artificial generativa como copiloto del desarrollo. El enfoque es dejar de hacer solo *vibecoding* descontrolado y, más bien, aplicar herramientas generativas de IA a la ingeniería de software. Los equipos deben poder explicar cada decisión técnica que tomaron, independientemente de qué herramienta generó el código.   
   2. La temática será revelada el día del evento a las 09:30 AM durante el Kickoff. Todos los equipos recibirán el mismo desafío al mismo tiempo.  
   3. Se revisará historial de *commits*

---

 

4. ## **Entregables:** 

   Los equipos deberán cumplir con los siguientes hitos durante el evento:

	**Hito 1 \- Prototipo de arquitectura** *Límite sugerido: 11:30 AM*

Durante el Sprint 0 (10:45 AM — 11:30 AM) solo se permite escribir código para realizar un prototipo de arquitectura. Este bloque está destinado a diseñar la arquitectura y documentar el proyecto. Al finalizar, cada equipo debe tener en su repositorio público de GitHub los siguientes archivos:

1. **README.md** con:  
* Nombre del proyecto y descripción breve (máximo 3 líneas)  
* Problemática que resuelve y usuario al que va dirigido  
* Stack tecnológico  
* Instrucciones para correr el proyecto localmente  
* Modelos y herramientas de IA que utilizarán  
* Nombres de los integrantes y roles definidos dentro del proyecto  
* Enlaces a documentación adicional  
2. **Diagramas de arquitectura y/o diseño** en formato PDF o .md (mermaid) dentro de la carpeta `/docs`, mostrando los módulos principales del sistema y cómo se comunican entre sí.  
3. **Otra documentación relevante (opcional):** modelos de datos, diagrama de despliegue, prompts relevantes utilizados, etc.  
   **Obligatorio:** Lanzar como *release* de Github el repositorio con el prototipo de arquitectura y toda la documentación incluida/anexada; nombrar el título del release como “Prototipo de arquitectura \- \[Nombre del proyecto\]”. Esto servirá para la evaluación posterior.  
   **Hito 2 — Entrega final** *Límite: 18:00 PM*  
   Cada equipo deberá lanzar una *release* de Github con los siguientes elementos:  
* **README.md actualizado** que refleje el estado final del proyecto y contenga **índices/anexos de interés** para el resto de documentación.  
* **Código de producción** completo  
* **Decisiones más relevantes** de diseño y arquitectura documentadas. Se recomienda el uso de ADRs en `/docs/adr/`  
* **URL de producción funcional** desplegada en la plataforma de su preferencia (Ej. Vercel, el timestamp del deploy en Vercel sirve como verificación complementaria).   
* **Pruebas Automatizadas (Testing Core):** El repositorio debe incluir al menos una suite de pruebas (Unit o Integration tests, ej: Jest, PyTest, JUnit) que valide el camino feliz (happy path) y un caso de error de la funcionalidad más crítica de su sistema. No se exige un % de cobertura alto, sino demostrar la capacidad de validar el código generado por IA de forma automatizada.

	Nombrar el título del release como “Entrega final \- \[Nombre del proyecto\]”

La aplicación debe estar operativa y accesible públicamente al momento de la evaluación, para ello colocar el enlace público anexado al repositorio de Github. No se aceptan demos pregrabados ni presentaciones en sustitución de la URL funcional. Si se realiza una app móvil, puede entregarse el apk y un video de demostración.

El equipo deberá preparar diapositivas para su presentación.

**Hito 3 — Pitch *18:00 PM — 19:15 PM***

Cada equipo tiene 5 minutos de presentación y 2 minutos de preguntas del jurado sobre cualquier parte del código, los cuales son parte de la evaluación. Durante el pitch el equipo debe:

* Explicar el problema que identificaron y por qué importa  
* Demostrar la solución en vivo desde la URL de producción  
* Sustentar las decisiones técnicas principales del proyecto

**Rúbrica de evaluación**

| Criterios | Pesos (%) |
| :---- | :---- |
| Arquitectura e Ingeniería | 40 |
| Funcionalidad y Despliegue | 30 |
| UX/UI y adaptación creativa | 15 |
| Pitch y Caso de Negocio | 15 |

---

5. ## **Premios:**

**Primer lugar — Premio "Cyborgs"** 

Premio hardware \+ plan de IA \+ premio digital \+ certificado de reconocimiento, para todo el equipo

**Segundo lugar — Premio "Hustlers"** 

Premio físico \+ premio digital \+ certificado de reconocimiento

**Premio especial sorpresa — "Chaotic Good"**

Para el equipo que enfrentó el colapso técnico más grande, o para el que puso más sobreingeniería sin quererlo, o para el que usar la IA fue más bien caótico, o para el que tuvo más potencial... Lo sabremos ese día.

---

6. ## **Restricciones, reglas y uso adecuado de IA:**

   1. ### **Restricciones y reglas:**

      1. Está **permitido** el uso de cualquier modelo de IA disponible públicamente, sin restricción de proveedor  
      2. Está **permitido** el uso de cualquier herramienta de desarrollo asistida por IA  
      3. Los jueces pueden solicitar explicaciones técnicas sobre cualquier parte del código durante la evaluación. El equipo es responsable de entender lo que construyó  
      4. Las entregas (*releases*) finales fuera de horario no serán consideradas, se revisará la hora exacta de subida. Solo se considerará el último envío realizado por el grupo antes de la hora final.  
      5. Los horarios establecidos podrán presentar ligeras variaciones debido a imprevistos, incidentes o situaciones operativas que puedan surgir durante el desarrollo del evento. Cualquier modificación será comunicada oportunamente a todos los participantes.

   2. ### **Uso adecuado de la IA:**

      1. **Para pensar:** Antes de escribir código, usarla para validar decisiones de arquitectura, comparar enfoques, o identificar casos borde que no habías considerado.  
      2. **Para acelerar lo tedioso:** Boilerplate, configuraciones, tests unitarios, tipado, documentación. Cosas que ya sabes cómo hacer pero que consumen tiempo sin agregar aprendizaje. Acá sí tiene sentido delegar casi completamente.  
      3. **Para aprender en contexto**: Cuando encuentras algo que no conoces, en lugar de solo pedirle que lo haga, pedirle que lo explique mientras lo construye. "Hazlo y explícame por qué funciona así."  
      4. **Para debuggear**: Darle el error y el contexto relevante y pedirle hipótesis, no soluciones directas. "¿Qué podría estar causando esto?" en lugar de "arréglalo". Así sigues siendo tú quien entiende el sistema. 

---

7. ## **Preguntas frecuentes**

   1. **¿Necesito experiencia previa en hackathons?** No. Se evalúa la calidad técnica del proyecto, no la experiencia en competencias previas.  
   2. **¿Puedo cambiar de equipo después de inscribirme?** No. Los equipos son fijos desde la postulación.  
   3. **¿Puedo usar una cuenta de pago en las herramientas de IA?** Sí, cada participante usa sus propias herramientas y cuentas.  
   4. **¿Qué pasa si Vercel falla al momento del deploy?** El equipo debe gestionar el deploy con tiempo suficiente antes del cierre. No se aceptan extensiones por problemas técnicos de plataformas externas.  
   5. **¿Puedo participar si no soy de la PUCP?** Sí, el evento está abierto a todo el público.  
   6. **¿Necesito llevar laptop?** Sí, cada integrante debe traer su propia laptop con su entorno de desarrollo configurado. El evento no provee equipos. Se recomienda llegar con las herramientas instaladas y listas para no perder tiempo del Sprint 0\.  
   7. **¿Puedo ausentarme durante el evento?** La asistencia presencial es obligatoria durante todo el evento. Ausencias prolongadas pueden ser motivo de descalificación a criterio de los organizadores.  
   8. **¿El evento cubre el costo de mis herramientas?** No. Cada participante utiliza sus propias cuentas y herramientas. Todas las herramientas permitidas cuentan con un plan gratuito disponible, por lo que no es requisito tener una cuenta de pago para competir

---

8. ## **Algunos recursos recomendados:**

- [https://c4model.com/](https://c4model.com/)   
- [https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns)  
- [https://adr.github.io/](https://adr.github.io/)  
- [https://dev.to/wallacefreitas/architecture-decision-records-adr-documenting-your-projects-decisions-5ac8](https://dev.to/wallacefreitas/architecture-decision-records-adr-documenting-your-projects-decisions-5ac8)   
- [https://www.youtube.com/watch?v=1CvXm-AjklQ](https://www.youtube.com/watch?v=1CvXm-AjklQ)   
- [Introducción a Jest (JavaScript/TypeScript)](https://jestjs.io/es-ES/docs/getting-started)  
- [Introducción a PyTest (Python)](https://docs.pytest.org/en/stable/)

---

¡AGRADECEMOS A NUESTROS SPONSORS\!

