# Plantilla para un spec útil

Este archivo es la referencia que consulta el skill `/spec` al generar specs. Cada sección incluye su propósito y un ejemplo mínimo. **No es texto para copiar literalmente** — es la forma que el skill debe respetar.

---

## Encabezado

Cada spec comienza con metadatos en formato blockquote (sin tablas, sin bloques, simple como se muestra abajo):

```markdown
# SPEC NN — Título corto y descriptivo

> **Estado:** Borrador
> **Depende de:** SPEC 01, SPEC 02
> **Fecha:** YYYY-MM-DD
> **Objetivo:** Una sola frase. Si necesitas dos frases, la feature es demasiado grande.
```

**Estados válidos:** `Borrador`, `En revisión`, `Aprobado`, `Implementado`, `Obsoleto`.

> Las etiquetas anteriores son las predeterminadas en español. Los skills también aceptan equivalentes en cualquier idioma (ej. inglés `Draft` / `In review` / `Approved` / `Implemented` / `Obsolete`). Elige un conjunto por repo y mantenlo consistente.

**Regla del objetivo:** una frase que un humano lee en 5 segundos y entiende qué se va a construir. Si no cabe en una frase, divide la feature.

---

## Sección 1 — Por qué existe este spec (opcional)

Para specs que toman decisiones no obvias o rompen patrones del proyecto, una breve sección que explique el **por qué** del trabajo. No el qué — el qué viene después.

Para specs simples, omítela.

---

## Sección 2 — Alcance

Dos sub-bloques explícitos. **Ambos son obligatorios.**

```markdown
## Alcance

**Entra:**

- Cosa concreta uno.
- Cosa concreta dos.

**Fuera de alcance (para specs futuros):**

- Algo que podría hacerse pero no ahora.
- Algo que surgió en la conversación pero no entra.
```

**Por qué importa el "fuera":** captura las cosas que el usuario mencionó durante la fase de preguntas pero se decidió posponer. Sin ese registro, durante la implementación habrá tentación de incluirlas "ya que estamos".

---

## Sección 3 — Modelo de datos

Las estructuras concretas que aparecen o cambian. Usa código real, no pseudocódigo abstracto.

```markdown
## Modelo de datos

\`\`\`ts
// Perfil de usuario
interface UserProfile {
  id: string;          // UUID v4
  nombre: string;
  email: string;
  rol: 'usuario' | 'admin';
  creadoEn: string;    // ISO 8601
}
\`\`\`

Convenciones:

- IDs: UUID v4.
- Fechas: ISO 8601 (`YYYY-MM-DDTHH:mm:ssZ`).
```

Si la feature no introduce datos nuevos, escríbelo explícitamente: _"Esta feature no introduce nuevas estructuras de datos. Reutiliza el modelo del SPEC 01."_

---

## Sección 4 — Plan de implementación

Pasos numerados. Cada paso debe dejar el sistema en un **estado funcional y ejecutable**. Sin "implementar la mitad y continuar mañana".

```markdown
## Plan de implementación

1. Crear el archivo X con un esqueleto vacío.
2. Implementar la función A en X. Prueba manual: ejecutar Y, ver Z.
3. Conectar X al módulo existente W.
4. ...
```

**Reglas:**

- Cada paso debe poder commitearse por sí solo.
- Si un paso requiere más de 30–50 líneas de código, divídelo.
- El último paso del plan **no** es "probar todo" — eso son los criterios de aceptación.

---

## Sección 5 — Criterios de aceptación

Checklist booleano. Cada item puede verificarse con sí o no.

```markdown
## Criterios de aceptación

- [ ] El formulario de registro muestra error si el email ya existe.
- [ ] Al iniciar sesión, el token se almacena y persiste tras recargar la página.
- [ ] Un usuario sin sesión que accede a /dashboard es redirigido a /login.
```

**Antipatrones a evitar:**

- ❌ "Que funcione bien." → no verificable.
- ❌ "Buena UX." → subjetivo.
- ❌ "Sin bugs." → no operacional.
- ✅ "Al enviar un formulario vacío se muestran mensajes de error bajo cada campo." → verificable, booleano.

---

## Sección 6 — Decisiones tomadas y descartadas

La sección con más valor dentro de 3 meses. Captura **qué consideraste**, no solo qué elegiste.

```markdown
## Decisiones

- **Sí:** JWT con refresh token en cookie httpOnly. Mitiga XSS sin perder persistencia de sesión.
- **No:** sessionStorage. Se pierde al cerrar la pestaña, mala UX.
- **Sí:** validación en cliente Y en servidor. El cliente da feedback rápido; el servidor es la fuente de verdad.
- **No:** OAuth por ahora. Va en otro spec si se decide agregar login social.
```

Cada decisión idealmente tiene una razón breve. Las decisiones sin razón son las primeras que se cuestionan después.

---

## Sección 7 — Riesgos identificados (opcional)

Solo cuando hay riesgos no obvios. Tabla simple:

```markdown
## Riesgos

| Riesgo                            | Mitigación                                                                          |
| --------------------------------- | ----------------------------------------------------------------------------------- |
| Token expirado durante la sesión  | Interceptor HTTP renueva el token automáticamente; si falla, redirige a /login.     |
| Servicio externo no disponible    | La API responde 503 con mensaje legible; el frontend muestra un banner de error.    |
```

Para specs pequeños o features muy contenidas, omítela.

---

## Sección final — Lo que NO entra (refuerzo)

Repite explícitamente al final qué **no** se hará en este spec. Esta repetición es deliberada — la sección de Alcance ya lo dice, pero al final del documento sirve de recordatorio cuando alguien lee solo las últimas líneas.

```markdown
## Lo que **no** entra en este spec

- Login con Google / proveedores OAuth (otro spec si se decide).
- Sincronización en tiempo real.
- Modo offline.

Cada uno de esos, si llega, va en su propio spec.
```

---

## Reglas globales sobre el documento completo

- **Una frase por idea.** Si una frase tiene dos comas y un punto y coma, divídela.
- **Nombres concretos.** Si dices "el módulo de usuarios", di `src/users/users.service.ts`. Si dices "una clave", da el string exacto.
- **Sin TODOs.** Un TODO en un spec significa que la decisión no se tomó. Tómala o regístrala como decisión pendiente con una razón.
- **Sin código ejecutable extenso.** El spec describe; el código se escribe después. Snippets cortos para ilustrar estructuras de datos están bien; funciones completas no.
- **Markdown estándar.** Sin extensiones raras. Debe renderizar en GitHub sin sorpresas.
