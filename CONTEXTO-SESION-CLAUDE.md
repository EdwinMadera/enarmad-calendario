# Contexto de sesión con Claude — Calendario ENARM (MED.AI)

> Documento de traspaso para conservar el contexto de esta conversación antes de
> formatear la computadora. **Guardado en el repositorio y subido a GitHub**, así
> que sobrevive al formateo: recupéralo con `git clone` o desde la web de GitHub.
>
> - **Repositorio:** `EdwinMadera/enarmad-calendario`
> - **Rama de trabajo:** `claude/affectionate-mendel-vns1r7`
> - **Fecha:** 2026-07-02

---

## 1. Qué es el proyecto

Aplicación web de un solo archivo (`index.html`, ~344 KB) que muestra un
**calendario de estudio para el ENARM**. Todo (HTML, CSS con Tailwind, JS y los
datos de los temas) vive embebido en ese único archivo. Los datos se guardan en
`localStorage` y opcionalmente se sincronizan con una "nube".

### Estructura de datos (interna del JS)
- `R.C[<calendario>].data` → arreglo de filas; cada fila es un arreglo de campos:
  - `r[1]` = **semana (`s`)**  ← campo central de esta auditoría
  - `r[3]` = tronco (índice a `R.T`)
  - `r[4]` = área
  - `r[5]` = tipo (`ti`: 0=Evidencias, 1=Flashcards, 2=Resúmenes, etc.)
  - `r[6]` = índice al nombre en `R.N`
- `R.N` = nombres de temas · `R.T` = troncos · `R.C` = calendarios
- `meta.fases` = fases (cada una con `semanas`, `color`, `nombre`)
- `meta.weeks` = metadatos por semana (fechas `start`/`end`)
- `items` = modelo de trabajo en memoria; cada ítem:
  `{id, s, d, tr, ar, ti, nm, pr, url, es, nota, done, ord}`
  - `s` = semana (1-based: la semana Nº N corresponde a `s === N`)
  - `d` = día (0=Lun … 6=Dom)
  - `ord` = orden dentro de la columna día
- Funciones clave: `buildSb` (barra lateral), `setW` (cambiar semana),
  `gP` (fase de una semana), `render`, `openMo`/`saveMo` (modal editar),
  `newC` (nueva tarjeta), `cloudSave`/`cloudLoad`, `load`.

---

## 2. Auditoría: hallazgos detectados

| # | Problema | Estado |
|---|----------|--------|
| 1 | **Ítems fantasma:** 376 temas con semana 0, invisibles en la UI; el progreso topaba en ~84% | ✅ **ARREGLADO** |
| 2 | "Restaurar todo" no es efectivo cuando la nube está activa (la nube vuelve a pisar los datos) | ⏳ Pendiente |
| 3 | Condición de carrera en `cloudSave` (guardados concurrentes pueden perder datos) | ⏳ Pendiente |
| 4 | Sobreescritura offline (trabajar sin conexión puede pisar la versión de la nube) | ⏳ Pendiente |
| 5 | Off-by-one de semana en el índice (`item.s+1` muestra `S(s+1)` en vez de `S(s)`) | ⏳ Pendiente |
| 6 | Tres calendarios vacíos | ⏳ Pendiente |
| 7 | Escapado de HTML (posible inyección al renderizar nombres/notas del usuario) | ⏳ Pendiente |

---

## 3. Detalle del hallazgo #1 (el que se arregló)

### Diagnóstico
- El calendario `"completo"` tenía **2298 ítems**, pero **376 (~16%)** tenían
  `s = 0` por un error de importación.
- La UI solo mostraba semanas **1..45**, así que esos 376 ítems eran
  **invisibles** y no se contaban por semana/fase, **pero sí** entraban en el
  total → la barra de progreso **nunca pasaba de ~84%**.
- Verificado: los 376 son **contenido único** (ningún duplicado de temas ya
  programados). Borrarlos habría perdido temas reales.
- Reparto de los 376 por tronco: Medicina interna **241**, Pediatría **69**,
  Cirugía **42**, GyO **24**.

### Solución implementada (no destructiva)
1. **Bucket "⚠ Sin asignar" (semana 0)** en la barra lateral (`buildSb`),
   navegable con `setW(0)`. Los 376 ítems quedan **visibles, contables y
   marcables** → el progreso ya puede llegar a **100%**.
2. **Selectores de Semana y Día** en el modal de edición (Semana `0` = sin
   asignar), con validación de rango `[0..total de semanas]` para que no se
   vuelvan a "perder". Permite reubicar cada tema a su semana/día reales.
3. `openMo`/`saveMo` cargan y aplican `s`/`d`; `saveMo` recalcula `ord` en el
   destino y refresca la barra lateral.

### Verificación (pruebas headless, sin navegador)
- 376 ítems ahora alcanzables → `bucket + semanas == total` (2298) ✔
- `setW(0)` no rompe nada y muestra los 376 ✔
- Al marcar todo como hecho → progreso **100%** ✔
- Reubicar un tema desde el modal lo saca del bucket (376 → 375) ✔

### Commit
- `0ad171c` — "Recuperar 376 temas «sin asignar» (semana 0) y permitir reubicarlos"
- Archivo modificado: `index.html`
- Pusheado a la rama `claude/affectionate-mendel-vns1r7`

---

## 4. Cómo continuar (próximos pasos sugeridos)

1. **Abrir el Pull Request** de la rama `claude/affectionate-mendel-vns1r7`
   (aún no se ha creado; requiere pedirlo explícitamente).
2. Atacar los hallazgos pendientes **#2–#7** (empezar por los de pérdida de
   datos: #3 carrera en `cloudSave`, #4 sobreescritura offline, #2 "Restaurar
   todo").
3. El off-by-one #5 es un arreglo trivial (`item.s+1` → `item.s`), pero conviene
   revisar antes que no afecte a los ítems transversales.

---

## 5. Recuperar este contexto tras formatear

```bash
git clone https://github.com/EdwinMadera/enarmad-calendario.git
cd enarmad-calendario
git checkout claude/affectionate-mendel-vns1r7
# Este archivo estará en la raíz: CONTEXTO-SESION-CLAUDE.md
```

Cuando retomes con Claude, comparte este archivo para restaurar todo el contexto.
