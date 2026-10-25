# Instrucciones de estructura — Plantilla de desarrollo IA (n8n)

> Este archivo lo deja el flujo cuando la carpeta aún no está inicializada.
> Dáselo a una IA de altas capacidades (Claude) junto a ti para generar el
> roadmap y el plan de trabajo **en el formato exacto que el pipeline espera**.
> El flujo de n8n NO inventa tareas: solo ejecuta lo que quede en `plan-activo/`.

## 1. Estructura de carpetas esperada

```
<proyecto>/
  documentacion/
    base/            objetivo-final.md · reglas.md · arquitectura.md
    roadmap/         roadmap.md  (visión por fases)
    entrada/         issues, ideas, borradores SIN curar  (el flujo NO lee aquí)
    plan-activo/     backlog.json  ← ÚNICA fuente de tareas ejecutables
      hechos/        (docs por tarea aprobada; los crea el flujo)
    issues/          (el arquitecto puede registrar bloqueos aquí)
    logs/            (rastro en vivo de cada IA; lo crea el flujo)
```

## 2. Qué debe contener `base/` (contexto del arquitecto)

- **objetivo-final.md**: qué logra el producto, qué NO es (alcance fuera), criterio de éxito.
- **reglas.md**: reglas obligatorias para toda IA (estilo, límites, qué no tocar, decisiones técnicas duras).
- **arquitectura.md**: stack DECIDIDO (lenguaje, almacenamiento, runner de pruebas). ⚠️ Si el lenguaje queda "a definir", el arquitecto rechazará las tareas: decídelo aquí antes de ejecutar.

## 3. `roadmap/roadmap.md`

Visión por fases: QUÉ y en qué orden. Cada fase se desglosa luego en tareas concretas del backlog. El roadmap es referencia, no se ejecuta directo.

## 4. `plan-activo/backlog.json` — EL formato que el flujo lee

Array JSON. El **Selector toma la PRIMERA tarea con `estado: "pendiente"`**.

```json
[
  {
    "id": "t002",
    "titulo": "Nombre corto de la tarea",
    "estado": "pendiente",
    "depende_de": ["t001"],
    "detalle": "Instrucción autocontenida y concreta: qué implementar, campos, validaciones, y qué queda FUERA de alcance. Referir a documentacion/base/ cuando aplique.",
    "criterios": [
      "Criterio verificable 1 (el arquitecto lo revisa uno por uno)",
      "Criterio verificable 2"
    ]
  }
]
```

**Campos:**
- `id`: string único y corto (`t001`, `t002`, …). El flujo lo usa para nombres de log, commits y docs.
- `titulo`: breve, aparece en el commit.
- `estado`: uno de `pendiente` · `en_progreso` · `hecho` · `revisar`. Tú creas tareas en `pendiente`. El flujo las marca `hecho` al aprobar, o `revisar` si no las pudo aprobar.
- `detalle`: la instrucción. Cuanto más clara y acotada, mejor ejecuta la IA. Una tarea = una unidad commiteable.
- `depende_de` *(opcional)*: lista de `id` que deben estar en `hecho` antes de ejecutar esta tarea. El Selector salta las pendientes con dependencias sin cumplir.
- `criterios` *(opcional)*: lista de criterios de aceptación verificables. El Arquitecto los revisa **uno por uno** para dar el veredicto.

**Reglas del backlog:**
- Solo lo que esté en `pendiente` se ejecuta. Si no hay ninguna `pendiente`, el flujo termina.
- Una tarea por cambio temático (no mezclar). Ordénalas por dependencia.
- No pongas tareas a medio pensar: lo borrador va en `entrada/`, no aquí.

## 5. Cómo trabaja el flujo (para que escribas tareas compatibles)

1. Selector elige 1 tarea `pendiente` y arma el prompt del ejecutor.
2. Ejecutor implementa SOLO esa tarea.
3. Evaluador revisa y corrige fallas obvias.
4. Arquitecto aprueba contra `base/` y el `detalle`; si aprueba → commit + doc + `hecho`; si no → reintenta y, si persiste, `revisar`.
5. Vuelve al Selector con la siguiente. Termina al no quedar `pendiente`.

Todo el trabajo ocurre en una rama `ia/*`; tú luego mergeas a `desarrollo` revisando los commits.

## 6. Tarea para la IA que lee esto

Con el usuario, y usando este formato:
1. Crea `base/objetivo-final.md`, `base/reglas.md`, `base/arquitectura.md` (con stack DECIDIDO).
2. Crea `roadmap/roadmap.md`.
3. Crea `plan-activo/backlog.json` con las primeras tareas en `pendiente`, bien acotadas.
4. No ejecutes nada: eso lo hará el flujo de n8n al correr.

## 7. Opciones del flujo (nodo Config en n8n)

Estos interruptores se ajustan en el nodo **Config** del workflow, no aquí:
- `arquitecto_usa_claude` (true/false): `true` usa Claude CLI como arquitecto (con respaldo a opencode si falla); `false` usa opencode con `modelo_arquitecto`. Cambia el CLI, no solo el modelo.
- `tiene_tests` + `comando_tests`: si `true`, el gate corre las pruebas; si fallan, se reintenta antes del arquitecto.
- `tiene_analisis` + `comando_analisis`: igual, para análisis estático (ej. PHPStan/Psalm).
- `tiene_compilacion` + `comando_compilar`: tras aprobar el arquitecto, compila/construye el proyecto; si falla, vuelve a desarrollo a corregir (lenguajes compilados: build real; interpretados: lint/build).
- El gate también rechaza automáticamente si se tocan carpetas protegidas (`base/`, `roadmap/`, `entrada/`).
