# Cómo escribir una buena tarea de backlog

La calidad del resultado es, como máximo, la calidad de la tarea. Antes de
poner una tarea en `backlog.json` como `pendiente`, revisa esta lista.

## Checklist (una tarea bien escrita cumple TODO esto)

- [ ] **Una sola unidad commiteable.** Si no cabe en un commit temático, pártela.
- [ ] **Autocontenida.** El `detalle` dice qué hacer sin suponer contexto que no esté en `base/` o `estado.md`.
- [ ] **Verificable.** Tiene `criterios` concretos y comprobables (no "que funcione bien").
- [ ] **Alcance cerrado.** Dice explícitamente qué queda FUERA.
- [ ] **Dependencias declaradas.** Si necesita otra tarea hecha, usa `depende_de`.
- [ ] **Testeable.** Se puede escribir una prueba que valide cada criterio.
- [ ] **No ambigua.** Nada de decisiones abiertas ("elige tú el enfoque"): eso se decide antes, en `base/` o contigo.

## Antipatrones (evítalos)

- ❌ Tareas "épicas" ("implementar el módulo de ventas"). → Pártelas en 3-6 tareas.
- ❌ Criterios vagos ("rápido", "limpio", "robusto"). → Hazlos medibles.
- ❌ Decisiones de arquitectura dentro de una tarea de implementación. → Van en `base/arquitectura.md`.
- ❌ Dos cambios temáticos en una tarea. → Sepáralos.

## Formato

```json
{
  "id": "t003",
  "titulo": "Verbo + objeto corto",
  "estado": "pendiente",
  "depende_de": ["t001"],
  "detalle": "Qué implementar, con campos/firmas concretas. Qué NO tocar.",
  "criterios": [
    "Dado X, cuando Y, entonces Z (comprobable con una prueba)",
    "No se modifica <archivo/comportamiento> existente"
  ]
}
```

## Regla de oro
Si tú no sabrías cómo verificar que la tarea quedó bien, la IA tampoco. Afínala
hasta que los `criterios` sean una prueba que cualquiera podría correr.
