# Plan de mantenimiento — simulatuhipotecario.cl

Sitio estático (`index.html`). Algunos datos envejecen con el mercado y la
normativa; este documento define **qué revisar, dónde, con qué fuente y cada
cuánto**, y el **estándar de proceso** para mantenerlo con la misma calidad.

## Estándar (report-and-approve)

Nada sale a producción sin aprobación. El proceso de cada revisión es:

1. **Leer** los valores actuales en `index.html` (objeto `TASAS`, array `BENEFICIOS`, tooltips, footer).
2. **Investigar** con el workflow multi-agente `.claude/mantenimiento-workflow.js` (fuentes oficiales, orientado a Chile y a la fecha vigente).
3. **Verificar de forma adversarial** cada cambio propuesto (que el dato sea real y citado; descartar alucinaciones).
4. **Informe de diffs**: por cada ítem → valor en código vs encontrado, ¿cambia?, valor propuesto, fuente, confianza, ubicación exacta.
5. **Aprobación del usuario** (este paso no se omite).
6. **Editar** `index.html`, **verificar en preview** (`preview_eval`, consola, responsivo) y **commit** con la convención de abajo → push → deploy automático.
7. Actualizar la etiqueta "actualizado" correspondiente y agregar una línea al **changelog** de este archivo.

## Cómo dispararlo

- Dile a Claude: **"corre el mantenimiento"** (o un ítem puntual, p. ej. "revisa solo las tasas").
- Claude lee los valores actuales, corre el workflow y entrega el informe para aprobar.
- **Agendado:** el `cron` de Claude Code se auto-expira a los 7 días y solo corre con Claude abierto — **no es un agendador permanente**. Lo confiable es dispararlo manualmente según la cadencia. Sugerencia: tasas ~mensual, subsidios ~trimestral, contribuciones en enero y julio.

## Qué mantener

| Dato | Dónde en `index.html` | Fuente oficial | Cadencia |
|---|---|---|---|
| Tasas (refs, típica, promedio) | objeto `TASAS` + footer "Parámetros a…" | [Banco Central](https://www.bcentral.cl/contenido/-/detalle/tasas-de-interes) + tasas de bancos | Mensual |
| Beneficios (topes UF, vigencia, cupos) | array `BENEFICIOS` | [MINVU](https://www.minvu.gob.cl), [ChileAtiende](https://www.chileatiende.gob.cl), gob.cl | Trimestral + por noticia |
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

## Convención de commits

- Commits normales (sin `--no-verify`). Mensaje en español neutro, descriptivo.
- Terminar el cuerpo con: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
- Push a `main` → Cloudflare Pages redespliega solo (sirve simulatuhipotecario.cl; calculatuhipotecario.cl redirige 301).

## Changelog de mantenimiento

<!-- Una línea por revisión: AAAA-MM-DD — qué se revisó y qué cambió. -->

- 2026-06-21 — Plan de mantenimiento creado. Línea base: tasas jun 2026 (típica 5,0%, promedio 4,06%), beneficios verificados vs MINVU/ChileAtiende/SII, monto exento $60.030.710.
