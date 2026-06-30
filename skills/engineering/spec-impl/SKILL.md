---
name: spec-impl
description: Implementa un spec aprobado. Valida que el estado signifique "Aprobado" (en cualquier idioma), crea una rama git con el nombre del spec, se mueve a ella, e inicia la implementación paso a paso con pausas para revisar diffs.
disable-model-invocation: true
argument-hint: <NN-nombre-del-spec>
allowed-tools: Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(cat:*), Bash(ls:*)
---

# /spec-impl — Implementador de specs aprobados

## Contexto de la sesión

Estado actual del repositorio:
!`git status --short`

Rama actual:
!`git branch --show-current`

Specs disponibles en esta carpeta:
!`ls specs/ 2>/dev/null || echo "La carpeta specs/ no existe"`

Configuración de creación de ramas:
!`cat specs/.spec-config.yml 2>/dev/null || echo "AutoCreateBranch: true (defecto, sin archivo de config)"`

---

## Instrucciones

Sigue estas cuatro fases en orden estricto. **No avances a la siguiente fase si la anterior no se completó correctamente.**

---

### Fase 1 — Identificar el spec

El argumento recibido es: `$ARGUMENTS`

Si `$ARGUMENTS` está vacío:

- Lista los archivos disponibles en `specs/` (ya los tienes arriba).
- Pídele al usuario que especifique el nombre exacto del spec.
- Detente y espera una respuesta. No continúes.

Si `$ARGUMENTS` tiene un valor:

- Busca el archivo en `specs/`. El usuario puede haber escrito el nombre completo (`01-mvp-arkanoid`), solo el número (`01`), o solo el slug (`mvp-arkanoid`). Intenta encontrar el archivo correcto en cualquiera de esos casos.
- Si no encuentras el archivo, muestra los specs disponibles y pídele al usuario que corrija el nombre.
- Si lo encuentras, continúa a la Fase 2.

---

### Fase 2 — Validar el estado del spec

Lee el archivo del spec que localizaste en la Fase 1 usando la herramienta Read o `cat`.

En el contenido del archivo, busca la línea que contiene el estado del spec. La etiqueta del encabezado suele ser `**Estado:**` (español) o `**Status:**` (inglés), pero puede estar en cualquier idioma. Identifícala por posición (línea de estado cerca del inicio del spec) y por la máquina de estados circundante, no por la etiqueta exacta.

**Regla absoluta:** Solo puedes continuar si el estado **significa "Aprobado"** — independientemente del idioma utilizado.

Trata cualquiera de los siguientes (y sus equivalentes en otros idiomas) como el estado **Aprobado** y continúa:

- Español: `Aprobado`
- Inglés: `Approved`
- Portugués: `Aprovado`
- Francés: `Approuvé`
- Alemán: `Genehmigt`
- Italiano: `Approvato`
- …o cualquier otra palabra en otro idioma que claramente signifique "aprobado"

Cualquier otra cosa (Borrador / Draft, En revisión / In review, Implementado / Implemented, Obsoleto / Obsolete, o cualquier valor no reconocido) significa **detenerse** y mostrar el mensaje de error que sigue.

| Categoría de estado                                  | Ejemplos (cualquier idioma)                       | Acción                                                                   |
| ---------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------ |
| Aprobado                                             | `Aprobado`, `Approved`, `Aprovado`, `Approuvé`, … | Continuar a la Fase 3.                                                   |
| Borrador                                             | `Borrador`, `Draft`, …                            | Detenerse. Mostrar el mensaje de error.                                  |
| En revisión                                          | `En revisión`, `In review`, …                     | Detenerse. Mostrar el mensaje de error.                                  |
| Implementado                                         | `Implementado`, `Implemented`, …                  | Detenerse. Mostrar el mensaje de error.                                  |
| Obsoleto                                             | `Obsoleto`, `Obsolete`, …                         | Detenerse. Mostrar el mensaje de error.                                  |
| Línea de estado no encontrada / valor no reconocido  | —                                                 | Detenerse. El archivo no sigue el formato esperado. Informar al usuario. |

Si no estás seguro de si un valor significa "aprobado", **no asumas**. Detente y pídele al usuario que aclare o que actualice el spec con la redacción canónica.

**Mensaje de error estándar cuando el estado no significa Aprobado:**

```
❌ No puedo implementar este spec.

Estado actual: [ESTADO ENCONTRADO]
Solo trabajo con specs cuyo estado signifique "Aprobado" (ej. `Aprobado`, `Approved`,
o el equivalente en otro idioma).

Para continuar tienes dos opciones:
  1. Si el spec está listo para implementarse, ábrelo y cambia el estado
     a "Aprobado" (o el término equivalente que use tu equipo) manualmente.
     Ese cambio lo hace el humano, no el agente.
  2. Si el spec todavía necesita trabajo, usa /spec [nombre] para retomarlo.
```

No ofrezcas alternativas, no sugieras "puedo empezar igual si quieres". El bloqueo es intencional.

---

### Fase 3 — Crear la rama git y moverse a ella

Una vez confirmado que el estado significa `Aprobado`:

1. Deriva el nombre de la rama del nombre completo del archivo del spec, sin la extensión. Formato gitflow: `feature/NN-slug`. Ejemplos:

   - `01-mvp-arkanoid.md` → rama `feature/01-mvp-arkanoid`
   - `02-powerups.md` → rama `feature/02-powerups`

2. Lee el flag `AutoCreateBranch` de la **Configuración de creación de ramas** mostrada en el contexto de sesión arriba.

   - Si el archivo de config no existe, falta el valor, o el valor no se reconoce → trátalo como `true` (el defecto).
   - Solo un `false` explícito (en cualquier capitalización) desactiva la creación automática de ramas.

   **Si `AutoCreateBranch` es `true` (defecto):** procede sin preguntar.

   - Si la rama **no existe**: créala con `git checkout -b feature/NN-slug`.
   - Si **ya existe**: informa al usuario que la rama ya existía (puede significar que se está retomando trabajo anterior).
   - En ambos casos: muévete a la rama con `git checkout feature/NN-slug` y confirma que el cambio fue exitoso antes de continuar.

   **Si `AutoCreateBranch` es `false`:** pregunta antes de tocar git. Muestra:

   ```
   AutoCreateBranch está en false.
   ¿Crear y moverse a la rama feature/NN-slug? [s/N]
   ```

   - Si el usuario responde **sí**: crea/muévete a la rama exactamente como en el caso `true` arriba.
   - Si el usuario responde **no** o lo deja vacío: **no crees ninguna rama.** Dile al usuario que implementarás en la rama actual (la mostrada en el contexto de sesión arriba) y pide confirmación explícita para continuar. No improvises — espera la respuesta.

3. Confirma visualmente al usuario que el spec está listo y qué rama está activa:

   ```
   ✅ Listo para implementar.

   Spec:   specs/NN-slug.md
   Rama:   feature/NN-slug  (activa)   (← o la rama actual, si no se creó una nueva)
   Estado: Aprobado   (← repite el valor exacto encontrado en el spec)
   ```

4. **No empieces a implementar todavía.** Primero muestra el resumen del spec al usuario para que lo tenga fresco. Extrae y muestra:
   - El **objetivo** (la línea después de `**Objetivo:**` / `**Objective:**` / etiqueta equivalente).
   - El **alcance** (la sección `## Alcance` / `## Scope` / equivalente).
   - El **plan de implementación** (la sección con los pasos numerados — `## Plan de implementación` / `## Implementation plan` / equivalente).
   - Los **criterios de aceptación** (el checklist — `## Criterios de aceptación` / `## Acceptance criteria` / equivalente).

Identifica los encabezados de sección por significado, no por redacción exacta — el spec puede estar en cualquier idioma.

---

### Fase 4 — Implementar paso a paso

Después de mostrar el resumen del spec, dile al usuario:

```
Voy a implementar el spec siguiendo el plan de implementación al pie de la letra.
Haré una pausa después de cada paso para que puedas revisar el diff.

¿Empezamos con el Paso 1?
```

Espera confirmación explícita ("sí", "adelante", "dale", o equivalente). No empieces sin ella.

Una vez confirmado, sigue estas reglas durante toda la implementación:

**Una regla sobre todas:** implementa lo que dice el spec. Si algo en el spec te parece subóptimo, menciónalo como observación pero implementa lo que se acordó. Los cambios al spec van al spec, no al código por sorpresa.

**Principios de implementación:**

- **Piensa antes de codificar.** Expón suposiciones, pregunta cuando no estés seguro, nunca adivines.
- **Simplicidad primero (Ponytail full).** El código mínimo que resuelve el problema. Sin abstracciones que nadie pidió. Antes de escribir, busca si ya existe o si de verdad hace falta.
- **Cambios quirúrgicos.** No tocar código no relacionado. Cada línea rastreable a lo pedido.
- **Metas verificables.** Convertir instrucciones vagas en criterios de éxito antes de escribir.
- **SOLID + Clean Code.** SRP por componente/servicio. Open/Closed: extender config maps, no switches en templates. Nombres que hablan.
- **Evaluar con las 10 heurísticas de Nielsen** todo lo que se haga; registrar hallazgos con la convención `H-xx`.
- **Verificar con Playwright** lo que aplique (credenciales por env; ver `e2e/`).
- **Herramientas en `.claude/`**: agentes (`ui-reviewer`, `api-consistency-reviewer`, `playwright-test-writer`) y skills (`gen-playwright-test`, `new-module`, `pr-check`). Skills de diseño: `angular-developer`, `frontend-design`, `mobile-design`, `ui-ux-pro-max`.
- **Por cada cosa terminada:** commit (co-author Claude) + actualizar `CONTEXT.md`.

**Ritmo de trabajo:**

- Implementa un paso del plan.
- Muestra un resumen de qué archivos tocaste y qué hiciste.
- Di: `Paso N completado. ¿Puedes revisar el diff y decirme si continúo con el Paso N+1?`
- Espera confirmación antes de continuar.

**Si durante la implementación encuentras una ambigüedad** que el spec no resuelve:

- Detente.
- Describe la ambigüedad exactamente.
- Presenta dos o tres opciones concretas.
- Espera la decisión del usuario.
- No improvises.

**Si el usuario pide algo que está fuera del alcance del spec:**

- Recuérdale que está fuera del alcance de este spec.
- Sugiere anotarlo para el próximo spec.
- No lo implementes en esta rama.

**Al terminar el último paso:**

```
✅ Todos los pasos del plan están implementados.

Siguiente paso: verifica los criterios de aceptación del spec uno por uno.
Si todos pasan, actualiza el estado del spec a "Implementado" (o el equivalente
en el idioma de tu repo) y haz el commit final antes de mergear esta rama.
```

---

## Resumen del comportamiento esperado

```
/spec-impl 01-mvp-arkanoid

  Fase 1  →  Encuentra specs/01-mvp-arkanoid.md
  Fase 2  →  Lee el estado → "Aprobado" (o "Approved", etc.) → ✅ continúa
  Fase 3  →  git checkout -b feature/01-mvp-arkanoid → git checkout feature/01-mvp-arkanoid
             Muestra objetivo, alcance, plan y criterios
  Fase 4  →  Implementa paso a paso con pausas
             Termina recordando verificar los criterios de aceptación

/spec-impl 02-powerups  (estado: Borrador / Draft)

  Fase 1  →  Encuentra specs/02-powerups.md
  Fase 2  →  Lee el estado → "Borrador" → ❌ se detiene
             Muestra el mensaje de error estándar
             No crea rama, no toca código
```

**La creación de ramas está controlada por el flag `AutoCreateBranch`** en `specs/.spec-config.yml`. Por defecto es `true` (crea la rama automáticamente, como se muestra arriba). Ponlo en `false` para que la Fase 3 pregunte `[s/N]` antes de crear la rama — útil si el nombrado de ramas es parte de tu propio flujo de Git.
