# Contexto de sesión con Claude — Calendario ENARM (MED.AI)

> **Documento de traspaso COMPLETO** para conservar todo el contexto antes de
> formatear la computadora. Guardado en el repositorio y **subido a GitHub**, así
> que sobrevive al formateo: recupéralo con `git clone` o desde la web de GitHub.
>
> - **Repositorio:** `EdwinMadera/enarmad-calendario` (GitHub)
> - **Rama de trabajo:** `claude/affectionate-mendel-vns1r7`
> - **Fecha:** 2026-07-02 · **Correo del usuario:** jemn884@gmail.com

> ⚠️ **Nota de seguridad:** este archivo NO contiene contraseñas ni claves
> secretas. La única URL de servicio listada (Google Apps Script) **ya es
> pública** porque está embebida en `index.html` dentro del repo. Aun así, si
> algún día quieres restringir el acceso, cambia el despliegue del Apps Script.
> **No** pegues aquí contraseñas de tus cuentas: guárdalas en un gestor de
> contraseñas.

---

## 0. CHECKLIST antes de formatear (para no batallar)

- [ ] **GitHub** — asegúrate de poder iniciar sesión en la cuenta `EdwinMadera`
      después del formateo (usuario/contraseña + 2FA/recovery codes). El código,
      esta documentación y todo el progreso viven ahí.
- [ ] **Cuenta de Google** — la sincronización en la nube depende de un *Google
      Apps Script* (ver §3). Guarda acceso a la cuenta de Google que es **dueña**
      de ese script; sin ella no podrás editar/re-desplegar el backend ni ver la
      hoja de datos.
- [ ] **Exporta un respaldo local** de tu progreso desde la app: botón
      **"Exportar JSON"** (genera `enarmad_completo.json`) y/o **"Exportar
      Excel"**. Guárdalo en la nube (Drive/GitHub), NO solo en el SSD.
- [ ] Verifica que el último push esté en GitHub (rama de arriba).
- [ ] (Opcional) Instala tras formatear: **Git**, un navegador y un editor
      (VS Code). No necesitas nada más para correr la app (es un solo HTML).

---

## 1. Qué es el proyecto

Aplicación web de **un solo archivo** (`index.html`, ~344 KB) que muestra un
**calendario de estudio para el ENARM**. Todo (HTML, CSS Tailwind, JS y los datos
de los temas) está embebido en ese archivo. El progreso se guarda en
`localStorage` del navegador y se sincroniza con una "nube" (Google Apps Script).

### Cómo correrla (tras formatear)
Es HTML puro: basta abrir `index.html` en el navegador. Requiere internet porque
carga librerías desde CDN (ver §4) y sincroniza con la nube.

### Estructura de datos (JS interno)
- `R.C[<calendario>].data` → filas; cada fila es un arreglo:
  `r[0]`=id, `r[1]`=**semana (`s`)**, `r[2]`=día (`d`), `r[3]`=tronco,
  `r[4]`=área, `r[5]`=tipo, `r[6]`=índice al nombre (`R.N`), `r[7]`=prioridad,
  `r[8]`=índice a URL (`R.U`), `r[9]`=estado.
- `R.N` = nombres · `R.U` = URLs · `R.T` = troncos · `R.C` = calendarios.
- `meta.fases` = fases (cada una con `semanas`, `color`, `nombre`).
- `meta.weeks` = fechas por semana (`start`/`end`).
- `items` = modelo en memoria: `{id,s,d,tr,ar,ti,nm,pr,url,es,nota,done,ord}`.
  - `s` = semana (**1-based**: semana Nº N ↔ `s === N`; `s === 0` = sin asignar).
  - `d` = día (0=Lun … 6=Dom) · `ord` = orden dentro de la columna.
- Variables globales por defecto: `cK='completo'` (calendario activo), `cW=1`
  (semana actual), `cP=0` (fase).

### Calendarios existentes (`R.C`)
| Clave | Ítems | Estado |
|-------|-------|--------|
| `completo` | **2298** | El único con datos |
| `ene_sep` | 0 | Vacío |
| `abr_sep` | 0 | Vacío |
| `ago_sep` | 0 | Vacío |

---

## 2. APIs, cuentas y almacenamiento (config real del código)

### 2.1 Backend de sincronización — Google Apps Script
- **`API_URL`** (en `index.html`, línea ~347):
  ```
  https://script.google.com/macros/s/AKfycbzpnZoNNPLpQaLQ0zOdnpcR7CilolK3hmM4q5Xs9Qlkm1k9M1CfLKX6nv8UK8is7rqW/exec
  ```
- Es un **Google Apps Script desplegado como Web App**. Actúa de backend de la
  nube. Casi seguro guarda los datos en una **Google Sheet** asociada.
- **Dueño:** la cuenta de **Google** con la que se creó el script (probablemente
  la tuya). Para editar/re-desplegar necesitas iniciar sesión en
  [script.google.com](https://script.google.com) con ESA cuenta.
- **Operaciones que usa la app:**
  - Guardar: `POST API_URL` con body JSON `{action:'saveProgress', cal, items}`
    (header `Content-Type: text/plain;charset=utf-8`).
  - Cargar: `GET API_URL?action=loadProgress&cal=<calendario>` → devuelve un
    arreglo de overrides.
- Funciones JS relacionadas: `cloudSave()`, `cloudLoad(k)`,
  `debouncedCloudSave()` (auto-guarda 2 s después de cada cambio),
  `upSyncBadge()` (indicador ☁ Sync/Error/Local).

### 2.2 Almacenamiento local (navegador)
- Progreso por calendario: clave `ea10_<calendario>` (p. ej. `ea10_completo`).
- Versión de datos: clave `ea10_ver_<calendario>` (invalida el caché local
  cuando cambian los datos base).
- **Ojo:** el `localStorage` es por-navegador y por-equipo → **al formatear se
  pierde**. Por eso el respaldo real es la nube (§2.1) y GitHub. Exporta JSON
  antes de formatear (§0).

### 2.3 Cuentas involucradas (resumen)
| Servicio | Para qué | Qué conservar |
|----------|----------|---------------|
| **GitHub** (`EdwinMadera`) | Código + esta doc + historial | Acceso a la cuenta (login + 2FA) |
| **Google** (dueño del Apps Script) | Backend/nube de sincronización | Acceso a esa cuenta de Google |
| Correo `jemn884@gmail.com` | Contacto del usuario | — |

> No hay otras API keys, tokens ni contraseñas en el código. Las demás
> dependencias son librerías públicas por CDN (§4).

---

## 3. Cómo funciona la sincronización (flujo `load`)

Al cargar un calendario (`load(k)`):
1. Decodifica los datos base embebidos (`dec(k)`).
2. Llama a `cloudLoad(k)` (la **nube es la fuente principal**).
   - Si la nube responde con datos → aplica overrides y **cachea** en
     `localStorage`.
   - Si la nube falla/está vacía → **fallback a `localStorage`**; si la versión
     cambió, borra el caché local.
3. `applyOverrides` fusiona por `id`: respeta `done`, `s`, `d`, `nm`, `url`, etc.,
   y agrega ítems nuevos (`isNew`).

Esto explica varios hallazgos (§5): si la nube siempre gana, "Restaurar todo"
(que borra el `localStorage`) no basta, porque la nube vuelve a imponer su
versión.

---

## 4. Dependencias externas (CDN)
- **SheetJS / xlsx** 0.18.5 — export/import Excel:
  `https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js`
- **Tailwind CSS** (runtime): `https://cdn.tailwindcss.com`
- **Google Fonts – Inter**: `https://fonts.googleapis.com/css2?family=Inter...`

---

## 5. Auditoría: hallazgos detectados

| # | Problema | Estado |
|---|----------|--------|
| 1 | **Ítems fantasma:** 376 temas con semana 0, invisibles en la UI; el progreso topaba en ~84% | ✅ **ARREGLADO** |
| 2 | "Restaurar todo" (`resetCal`) no es efectivo con la nube activa: borra `localStorage` pero la nube vuelve a pisar los datos | ⏳ Pendiente |
| 3 | Condición de carrera en `cloudSave` (el flag `syncing` puede descartar guardados; cambios rápidos se pierden) | ⏳ Pendiente |
| 4 | Sobreescritura offline: si la nube falla al cargar pero luego `save()` sube el estado local, puede pisar datos buenos de la nube | ⏳ Pendiente |
| 5 | Off-by-one de semana en el índice (`item.s+1` muestra `S(s+1)` en vez de `S(s)`) | ⏳ Pendiente |
| 6 | Tres calendarios vacíos (`ene_sep`, `abr_sep`, `ago_sep`) | ⏳ Pendiente |
| 7 | Escapado de HTML: nombres/URLs/notas del usuario se insertan sin escapar → posible inyección al renderizar | ⏳ Pendiente |

---

## 6. Detalle del hallazgo #1 (arreglado)

### Diagnóstico
- El calendario `"completo"` tenía **2298 ítems**, pero **376 (~16%)** con
  `s = 0` por un error de importación.
- La UI solo mostraba semanas **1..45**, así que esos 376 eran **invisibles** y
  no se contaban por semana/fase, **pero sí** entraban en el total → el progreso
  **nunca pasaba de ~84%**.
- Verificado: los 376 son **contenido único** (ningún duplicado). Borrarlos
  habría perdido temas reales.
- Reparto por tronco: Medicina interna **241**, Pediatría **69**, Cirugía **42**,
  GyO **24**.

### Solución implementada (no destructiva)
1. **Bucket "⚠ Sin asignar" (semana 0)** en la barra lateral (`buildSb`),
   navegable con `setW(0)` → los 376 quedan **visibles, contables y marcables**;
   el progreso ya puede llegar a **100%**.
2. **Selectores de Semana y Día** en el modal de edición (Semana `0` = sin
   asignar), con clamp al rango válido `[0..total de semanas]`. Permite reubicar
   cada tema a su semana/día reales.
3. `openMo`/`saveMo` cargan y aplican `s`/`d`; `saveMo` recalcula `ord` y refresca
   la barra lateral.

### Verificación (pruebas headless, sin navegador)
- 376 ítems ahora alcanzables → `bucket + semanas == total` (2298) ✔
- `setW(0)` no rompe nada y muestra los 376 ✔
- Marcar todo hecho → progreso **100%** ✔
- Reubicar un tema lo saca del bucket (376 → 375) ✔

### Commits en la rama `claude/affectionate-mendel-vns1r7`
- `0ad171c` — Recuperar 376 temas «sin asignar» (semana 0) y permitir reubicarlos
- `6a2d473` — Añadir documento de traspaso de contexto de la sesión
- (+ este documento actualizado con APIs/cuentas)

---

## 7. Registro de la conversación (resumen)

1. Auditoría del calendario ENARM → 7 hallazgos (tabla §5). El usuario eligió
   **arreglar el #1** (ítems fantasma) y dio libertad ("no preference") sobre qué
   hacer con los 376 ítems.
2. Claude confirmó que los 376 son **únicos** (no duplicados) → decisión: no
   borrarlos, sino recuperarlos con un bucket "Sin asignar" + reubicación.
3. Se implementó y **verificó headless** el arreglo #1. Commit `0ad171c`, pusheado.
4. El usuario mencionó desconectar el "MED.AI SSD de 1 TB" y que **formateará**
   la computadora → se aclaró que el entorno es remoto/efímero y que el respaldo
   real es GitHub.
5. El usuario pidió guardar **todo el contexto del chat** en un `.md`,
   **incluyendo APIs y cuentas**, para no batallar tras el formateo → este
   documento (commit de actualización).

---

## 8. Próximos pasos sugeridos

1. **Abrir el Pull Request** de `claude/affectionate-mendel-vns1r7` (aún no
   creado; requiere pedirlo explícitamente).
2. Atacar los pendientes por prioridad de pérdida de datos: **#3** (carrera en
   `cloudSave`), **#4** (sobreescritura offline), **#2** ("Restaurar todo").
3. **#5** off-by-one: cambio trivial (`item.s+1` → `item.s`), revisar antes que
   no afecte ítems transversales.
4. **#7** escapado de HTML: escapar `nm`/`url`/`nota` al renderizar.

---

## 9. Recuperar este contexto tras formatear

```bash
git clone https://github.com/EdwinMadera/enarmad-calendario.git
cd enarmad-calendario
git checkout claude/affectionate-mendel-vns1r7
# En la raíz: CONTEXTO-SESION-CLAUDE.md  ← este archivo
```

Cuando retomes con Claude, comparte este archivo para restaurar todo el contexto.
