# Plan de mantenimiento — simulatuhipotecario.cl

Sitio estático (`index.html`). Algunos datos envejecen con el mercado y la
normativa; este documento define **qué revisar, dónde, con qué fuente y cada
cuánto**, y el **estándar de proceso** para mantenerlo con la misma calidad.

## Estándar (report-and-approve)

Nada sale a producción sin aprobación. El proceso de cada revisión es:

1. **Leer** los valores actuales en `index.html`: objeto `TASAS`, array `BENEFICIOS`, tooltips, footer, las **constantes** (`ratio`, `FACTOR_HON`, `/0.45`, `edadFin>75`) y el **FAQ + JSON-LD**.
2. **Investigar** con el workflow multi-agente `.claude/mantenimiento-workflow.js` (fuentes oficiales, orientado a Chile y a la fecha vigente).
3. **Verificar de forma adversarial** cada cambio propuesto (que el dato sea real y citado; descartar alucinaciones).
4. **Informe de diffs**: por cada ítem → valor en código vs encontrado, ¿cambia?, valor propuesto, fuente, confianza, ubicación exacta.
5. **Aprobación del usuario** (este paso no se omite).
6. **Editar** `index.html`, **verificar en preview** (`preview_eval`, consola, responsivo) y **commit** con la convención de abajo → push → deploy automático.
7. Actualizar la etiqueta "actualizado" correspondiente y agregar una línea al **changelog** de este archivo.

> **Consistencia (importante):** varios datos financieros viven a la vez en el **cálculo** (constantes) y en **textos** (FAQ, JSON-LD, footer, tooltips, hints). Al cambiar uno, actualizar **todas** sus apariciones — y el **FAQ del JSON-LD debe quedar idéntico al FAQ visible**.

## Cómo dispararlo

- Dile a Claude: **"corre el mantenimiento"** (o un ítem puntual, p. ej. "revisa solo las tasas").
- Claude lee los valores actuales, corre el workflow y entrega el informe para aprobar.
- **Agendado:** el `cron` de Claude Code se auto-expira a los 7 días y solo corre con Claude abierto — **no es un agendador permanente**. Lo confiable es dispararlo manualmente según la cadencia. Sugerencia: tasas ~mensual, subsidios ~trimestral, contribuciones en enero y julio.

## Qué mantener

| Dato | Dónde en `index.html` | Fuente oficial | Cadencia |
|---|---|---|---|
| Tasas (refs, típica, promedio) | objeto `TASAS` + footer "Parámetros a…" | [Banco Central](https://www.bcentral.cl/contenido/-/detalle/tasas-de-interes) + tasas de bancos | Mensual |
| Beneficios (topes UF, vigencia, cupos) | array `BENEFICIOS` | [MINVU](https://www.minvu.gob.cl), [ChileAtiende](https://www.chileatiende.gob.cl), gob.cl | Trimestral + por noticia |
| **Supuestos/reglas** (ratio 25/30, carga 45%, honorarios ~70%, pie 20/10%, edad ≤75) | constantes (`ratio`, `FACTOR_HON`, `/0.45`, `edadFin>75`) **+ textos** (FAQ, JSON-LD, footer, tooltips) | [CMF Educa](https://www.cmfchile.cl/educa/) + bancos | Anual / al cambiar políticas |
| Monto exento contribuciones | tooltip "exención de contribuciones" | [SII](https://www.sii.cl) | Semestral (ene/jul) |
| Enlaces oficiales (que sigan 200) | `BENEFICIOS` (`url`) + footer + JSON-LD | — | Mensual |
| UF (valor de respaldo) | `FALLBACK` en `cargarUF()` | mindicador.cl (Banco Central) | Solo si la API falla seguido |
| Etiquetas de año | `<title>`, meta, footer, `og.svg` | — | Anual (enero) |

## Detalle por ítem

### 1. Tasas — objeto `TASAS`
Valores vigentes (jun 2026): `actualizado:'junio 2026'`, `promedioSistema:4.06`, `tasaDefault:5.00`, `refs:` Mejor caso 3,40 · Promedio 4,06 · Típico fijo 5,00 · Conservador 5,50.
Al actualizar: cambiar los `refs[]`, `tasaDefault`, `promedioSistema`, la etiqueta `TASAS.actualizado`, y el texto del footer ("Parámetros a <mes año>: … TPM … promedio del sistema … rango …"). Mantener la lógica (default = típica fija a 25–30 años).

### 2. Beneficios — array `BENEFICIOS`
Re-verificar para cada uno: tope(s) en UF, `estado` (`vigente`/`porconfirmar`), `soloNueva`, condiciones y la `nota` (vigencia/cupos). Verificar que cada `url` siga devolviendo 200.
**Eventos clave a vigilar:**
- **CEEC** (IVA constructoras): en 2026 al 25%, **se elimina el 1-ene-2027** → en 2027 quitar de la nota tributaria.
- **DS1 Tramo 4** (hasta 4.000 UF): parte 2.º sem 2026 sin reglamento → cuando se publique, pasar `estado` a `vigente` y precisar condiciones.
- **Subsidio al Dividendo / FOGAES** (Ley 21.748): cupos (50.000) casi agotados a abr-2026; el gobierno tramitaba duplicarlos a 100.000 → actualizar la `nota`. Plazo legal ~may-2027.
- **DS19**: vigencia 2026 por confirmar → revisar si hay llamado.
Los topes en UF de DS1/DS19/DS49 se reajustan; confirmar contra MINVU/ChileAtiende.

### 3. Monto exento contribuciones — tooltip "exención de contribuciones"
Valor vigente: **$60.030.710 (1.er sem. 2026)**. El SII lo reajusta cada semestre. Actualizar el número en el `aria-label` y en el `.tipbox`.

### 4. Enlaces oficiales
Verificar 200: las 7 URLs de `BENEFICIOS`, las del footer (bcentral, simulador CMF) y la del JSON-LD. Si una cambió de ruta, reemplazarla por la oficial vigente.

### 5. Etiquetas de año
En enero: revisar "2026" en `<title>`, `meta description`, footer ("Parámetros a … 2026"), `og.svg`/`og.png`, y JSON-LD. Actualizar al año vigente si corresponde.

### 6. Supuestos y reglas financieras (cálculo + textos)
Reglas que la banca/CMF puede cambiar y que están **a la vez en el cálculo y en varios textos** — al tocar una, hay que actualizar **todas** sus apariciones:
- **Ratio dividendo/renta 25% (estricto) / 30% (flexible)** — `ratio` (toggle), `compute()`, FAQ, JSON-LD, footer ("ratio 25% CMF Educa"), tooltips de "¿Te alcanza tu renta?".
- **Carga financiera total máx 45%** — `/0.45` en `compute()` (~líneas 980 y 1200), FAQ, JSON-LD, fila "Renta por carga total".
- **Honorarios castigo ~70%** — `FACTOR_HON = 0.70` (~línea 934), FAQ, tooltip "honorarios", hint de la tarjeta de renta.
- **Pie 20% / 10% con FOGAES** (financiamiento 80%/90%) — default del slider, hint de financiamiento, FAQ, footer.
- **Edad máx al término ~75** — `edadFin>75` (~línea 1383) + texto del aviso.
- **Seguros típicos** — defaults desgravamen 0,020% / incendio 0,025% (inputs).
Fuente: [CMF Educa](https://www.cmfchile.cl/educa/) y prácticas bancarias. Recordatorio: el **FAQ del JSON-LD debe quedar idéntico al FAQ visible**.

## Convención de commits

- Commits normales (sin `--no-verify`). Mensaje en español neutro, descriptivo.
- Terminar el cuerpo con: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
- Push a `main` → Cloudflare Pages redespliega solo (sirve simulatuhipotecario.cl; calculatuhipotecario.cl redirige 301).

## Changelog de mantenimiento

<!-- Una línea por revisión: AAAA-MM-DD — qué se revisó y qué cambió. -->

- 2026-06-21 — Plan de mantenimiento creado. Línea base: tasas jun 2026 (típica 5,0%, promedio 4,06%), beneficios verificados vs MINVU/ChileAtiende/SII, monto exento $60.030.710.
- 2026-06-21 — 1.ª revisión (workflow). Cambios aplicados: promedio del sistema 4,06% → **3,96%** (Banco Central, dato de mayo 2026) en `TASAS.promedioSistema`, ref "Promedio", tabla "Comparar tasas" y footer; corregida redacción de FOGAES ("60% del crédito" → "del valor de la vivienda", per gob.cl). Verificado vigente sin cambios: TPM 4,5%, tasas típica/mejor caso/conservador, DS49/DS1/DS19/Subsidio al Dividendo/CEEC, monto exento $60.030.710 y los 12 enlaces oficiales (HTTP 200).
