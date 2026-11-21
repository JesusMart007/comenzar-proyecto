# Prompt para el agente — Proyecto TallerYa

## Tu tarea

Actúa como ingeniero de software senior. Tu trabajo es preparar **solo el plan de desarrollo** de la aplicación que se describe abajo.

- Guarda el plan en **`PLAN.md`**, en la raíz del repositorio.
- **No escribas código, no crees carpetas del proyecto y no instales nada todavía.** Lo revisaré antes de empezar.
- Si algo es ambiguo o contradictorio, **no lo inventes**: añádelo a la sección "Preguntas para el cliente" del plan, con tu propuesta por defecto.
- Respeta las reglas de negocio exactamente como están escritas. **No añadas "alternativas"** a esas reglas.

---

## 1. Contexto del cliente

- El negocio se llama **TallerYa** y organiza talleres presenciales de cualquier temática (cerámica, programación, fotografía, mecánica, cocina, dibujo...).
- Hoy gestiona todo por **WhatsApp + Excel**, y eso le da estos problemas:
  - no sabe quién reservó primero;
  - hay reservas duplicadas;
  - se le pierden datos;
  - le cuesta avisar a todos cuando se cancela una clase;
  - no sabe qué talleres funcionan mejor;
  - no aparece en Google.
- Tiene **una sola sede** (un local alquilado en Bogotá). Todos los cursos son **presenciales**, de **un solo día** y de pocas horas (3–4 h).
- Hay tres tipos de usuario: el **dueño (admin)**, los **profesores** (con **salario fijo**, que paga el dueño) y los **estudiantes**.
- **Idioma:** toda la interfaz, los emails y los certificados van **solo en español**.
- **Moneda:** todos los importes van en **euros (EUR)**.

## 2. Aplicaciones

1. **Landing page pública**, llamativa y con buen SEO.
2. **Back office / plataforma**, con acceso según el rol.

- Dominio principal (p. ej. `tallerya.com`) para la web pública y un **subdominio** para la plataforma.
- Diseño **mobile-first** y totalmente responsive (móvil, tablet y escritorio).

## 3. Landing page

- Sección *hero* llamativa y una sección informativa sobre el negocio.
- **Catálogo** de los talleres publicados, con la dirección de la sede.
- **SEO fuerte**: metadatos, *sitemap* y datos estructurados de schema.org (tipo `Course`/`Event`).
- **Que la encuentren los agentes de IA**: contenido legible por máquinas, datos estructurados y `llms.txt`. Así, si alguien le pide a un asistente "talleres de fotografía en Bogotá", pueden salir los de TallerYa.
- Banner de consentimiento de cookies y analítica.

## 4. Roles y permisos

| Acción | Admin | Profesor | Estudiante | Público |
|---|:-:|:-:|:-:|:-:|
| Ver la landing y el catálogo | ✅ | ✅ | ✅ | ✅ |
| Crear, editar y activar/desactivar profesores | ✅ | — | — | — |
| Crear, editar y habilitar/inhabilitar cursos | ✅ | — | — | — |
| Editar **solo el contenido o material** de sus cursos y subir la URL de la grabación | ✅ | ✅ (solo los suyos) | — | — |
| Ver sus cursos, los alumnos inscritos y cuánto se están llenando | ✅ | ✅ (solo los suyos) | — | — |
| Marcar pagos como recibidos | ✅ | — | — | — |
| Panel de ganancias | ✅ | — | — | — |
| Ranking de talleres y analítica | ✅ | — | — | — |
| Inscribirse, cancelar, ver material y descargar el certificado | — | — | ✅ | — |

- Cada usuario solo ve lo que corresponde a su rol.
- El profesor **no** crea cursos ni ve las ganancias del negocio.

## 5. Usuarios

### Estudiantes
- Se **registran ellos mismos**. Necesitan tener **cuenta** para poder inscribirse a un taller.
- Campos: nombre, apellido, teléfono, email (único) y contraseña.
- Consentimientos en el registro, todos con la casilla **desmarcada por defecto** (marcarla por defecto es ilegal):
  - política de privacidad → **obligatoria**;
  - notificaciones sobre sus cursos (avisos, recordatorios, plazas libres de la lista de espera) → opcional;
  - promociones y ofertas → opcional.
- Junto a las casillas, un texto que anime a aceptar las notificaciones (por ejemplo: "si no aceptas, no podremos avisarte de plazas libres").
- Todos los emails llevan un enlace para **darse de baja** en un clic.

### Profesores
- **Solo el admin** puede darlos de alta y activarlos o desactivarlos.
- El admin les pone una contraseña inicial y el profesor puede **cambiarla** por la suya.
- Perfil: bio y teléfono (la foto es opcional).
- **Salario fijo** en euros, que solo ve y edita el admin. El profesor cobra lo mismo con independencia de los cursos o alumnos que tenga.

## 6. Cursos (los crea el admin)

- Campos:
  - título, descripción e imagen;
  - profesor asignado;
  - **precio** en euros (lo fija el admin);
  - **duración en horas** (siempre en un solo día);
  - fecha y hora de inicio;
  - **capacidad**, distinta para cada curso;
  - fecha límite de inscripción;
  - estado: borrador / publicado / cancelado / finalizado.
- Validaciones: la fecha límite de inscripción es anterior al inicio, el precio es mayor que 0 y la capacidad es mayor que 0.
- **Precio:** lo decide el admin directamente. No hay comisiones ni reparto con el profesor.

## 7. Reglas de negocio de las reservas (obligatorias)

1. **Una plaza por persona y por curso.** Implementarlo con una restricción única sobre las inscripciones activas. Un estudiante **sí** puede inscribirse a cursos distintos.
2. Al inscribirse, la reserva pasa a **`pending_payment`** y tiene un plazo de pago de **ahora + 24 h**.
3. **Sin pasarela de pago en esta fase.** Tras reservar, se muestra al estudiante el QR o los datos de transferencia bancaria. El estudiante envía el comprobante por WhatsApp y **el admin marca el pago** en el back office.
4. Si no hay pago en 24 h, un **worker/job** pasa la reserva a **`expired`** y libera la plaza.
5. El curso se considera **lleno con los N primeros que pagan**.
6. **Lista de espera:** cuando se libera una plaza, se avisa por email a los interesados que hayan aceptado las notificaciones.
7. **Ventana de inscripción:** se cierra **24 h antes** del inicio. Con menos de 24 h no se puede reservar, aunque queden plazas.
8. **Cancelación del estudiante:** si cancela **48 h o más antes** del inicio, se le devuelve el **100 %** (la devolución la hace el admin a mano). Si cancela con menos de 48 h, no hay devolución.
9. Si se **cancela un curso**, se avisa a todos los inscritos.
10. Estados de una inscripción: `pending_payment` → `paid` → `completed`, o bien `cancelled` / `expired`. Incluye en el plan el diagrama de estados.
11. Prepara una interfaz `PaymentProvider` para poder conectar más adelante Stripe o Redsys.

## 8. Área del estudiante

- Ver sus inscripciones y el estado de cada una.
- Ver el **material** del curso y la **grabación** (una URL de YouTube que sube el profesor).
- Descargar su **certificado en PDF**. Se genera cuando la inscripción está pagada y el curso ha terminado; basta con haber asistido.

## 9. Panel de ganancias (solo admin)

- Ingresos cobrados (inscripciones pagadas), por curso y por periodo.
- Coste de los **salarios fijos** de los profesores en ese periodo.
- **Beneficio neto** del negocio: ingresos menos salarios.

## 10. Telemetría y analítica (solo admin)

- Qué cursos se visitan y se buscan más.
- **Embudo de ventas** (visitas → registros → inscripciones → pagos) y **tasa de abandono**.
- **Ranking** de los talleres más populares y de los que se llenan antes.
- Analítica propia, con consentimiento y sin guardar datos personales innecesarios.

## 11. Stack (obligatorio)

- **Frontend:** Next.js (App Router) + **TypeScript** + **Tailwind CSS**.
- **Backend:** Python + **FastAPI** + Pydantic + SQLAlchemy + **Alembic** (migraciones versionadas).
- **Base de datos:** PostgreSQL.
- **Autenticación:** **JWT**, con un token de acceso de corta duración y un *refresh token*.
- **Docker + Docker Compose**, con servicios separados para base de datos, backend, frontend, worker y email. Usa la **última versión estable** de cada imagen y librería (no la más reciente) y fija las versiones.
- **Dependabot** configurado para avisar de dependencias desactualizadas o vulnerables.
- **Arquitectura por capas:** API / servicios / dominio / acceso a datos.

## 12. Testing

- **Test primero**: escribir los tests antes de cada funcionalidad.
- Cubrir el **camino feliz**, los **casos límite** y los **errores**. Como mínimo:
  - la plaza única por curso;
  - el aforo completo;
  - la caducidad a las 24 h sin pago;
  - el cierre de inscripción 24 h antes del curso;
  - la cancelación con más y con menos de 48 h;
  - la lista de espera;
  - los permisos de cada rol.
- Tipos de test: unitarios, de integración, **E2E con Playwright** y **visuales** en varios dispositivos (p. ej. iPhone 13 y Pixel 7).

## 13. Forma de trabajo

- Desarrollo **por fases**, nunca todo de golpe. En cada fase: modelos → endpoints → frontend → tests → mi verificación.
- **Git:** la rama por defecto es `development`. Cada fase o funcionalidad va en **su propia rama**, creada a partir de `development`.
- **Commits:** los haces tú, pero **solo después de que yo apruebe** el resultado de la fase. El push lo hago yo.
- Al final de cada fase, deja un archivo en `docs/learning/` que explique los conceptos técnicos usados y por qué se eligieron.
- Para el diseño visual te pasaré capturas de webs de referencia. No hagas un diseño genérico.

---

## Qué debe incluir `PLAN.md`

1. **Resumen y alcance**: qué entra en el MVP y qué queda para después.
2. **Modelos y tablas** con todos sus campos, tipos, claves primarias y foráneas, enums y restricciones. Este es el primer punto que revisaré con el cliente.
3. **Diagrama de estados** de la inscripción.
4. **Arquitectura** y **estructura de carpetas** del repositorio.
5. **Servicios de Docker Compose** y las versiones elegidas.
6. **Endpoints** de la API, agrupados por recurso y con el rol que necesita cada uno.
7. **Estrategia de testing** y lista de casos que se van a cubrir.
8. **Seguridad**: JWT, contraseñas, permisos y RGPD/consentimientos.
9. **Fases de desarrollo**, cada una con su rama, sus entregables y su criterio de "terminado". Orientativas:
   - **Fase 0** — Base: Docker, esqueleto, CI y Dependabot
   - **Fase 1** — Base de datos y autenticación (registro y login funcionando en el frontend)
   - **Fase 2** — Gestión de usuarios y profesores
   - **Fase 3** — Cursos
   - **Fase 4** — Landing, SEO e inscripciones (reservas, pagos manuales, caducidad y lista de espera)
   - **Fase 5** — Paneles del back office, ganancias y certificados
   - **Fase 6** — Telemetría y analítica
10. **Preguntas para el cliente**: cualquier duda que te surja, cada una con tu propuesta por defecto.

---

## Fuera de alcance (NO incluir)

- Importar cursos desde CSV.
- Cursos online, cursos de varios días o varias sedes.
- Pasarela de pago real, y pagos en efectivo o mixtos.
- Reservar varias plazas desde una misma cuenta.
- "Una sola plaza activa en toda la plataforma" (la regla es una plaza **por curso**).
- Cancelar "48 h después de inscribirse" y devoluciones parciales o automáticas.
- Cálculo de gastos o del punto de equilibrio de un curso.
- Que el profesor cree cursos o vea las ganancias del negocio.
- Comisiones, reparto de ingresos o pago al profesor por curso (el profesor cobra un salario fijo).
- Multidioma (solo español) y multimoneda (solo EUR).
- Casillas de consentimiento marcadas por defecto.
