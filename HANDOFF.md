# calculatuhipotecario.cl — Handoff

Contexto y tareas pendientes migradas desde una sesión de planificación. El objetivo es publicar una calculadora hipotecaria estática en el dominio propio.

## Qué es

Calculadora de **renta líquida mínima requerida** para acceder a un crédito hipotecario en Chile, según el valor de una propiedad. 100% estática: un solo archivo HTML con CSS/JS inline, sin backend ni base de datos. Archivo principal: `index.html` (mantener ese nombre — el host sirve `index.html` en la raíz).

## Setup objetivo

- **Dominio:** `calculatuhipotecario.cl` (registrado en NIC Chile).
- **Host:** Cloudflare Pages (gratis, estático, deploy automático conectado a Git).
- **Repo:** GitHub (a crear).
- **Flujo final:** editar con Claude Code → push a GitHub → Cloudflare redespliega solo.

## Parámetros ya incorporados en la calculadora (Chile, jun 2026)

- Ratio dividendo/renta: 25% (estricto) / 30% (flexible).
- Tope carga financiera total: 45% (dividendo + otras deudas).
- Financiamiento estándar 80% (pie 20%); hasta 90% solo con FOGAES/Subsidio Dividendo, vivienda nueva ≤ UF 4.000.
- Tasa default 4,5% (conservadora); referencias de mercado: Itaú 3,39%, promedio 4,06%, BancoEstado 4,19%.
- UF y tasa son inputs editables. La UF tiene default hardcodeado (40793) → ver mejora pendiente A3.
- Amortización francesa, tasa mensual = anual ÷ 12.

---

## Pendiente

### A. Lo que Claude Code SÍ puede hacer (en esta sesión)

1. `git init` del proyecto, con `index.html` en la raíz.
2. Crear el repo en GitHub y hacer el primer push. **Requiere** `git` + autenticación de GitHub en el terminal (lo más simple: `gh auth login` una vez; instalar GitHub CLI con `winget install GitHub.cli` si no está).
3. **[Mejora]** Agregar fetch de la UF en vivo desde mindicador.cl al cargar la página, con fallback al valor hardcodeado, e indicador "UF en vivo / respaldo". Spec abajo.
4. Commit + push de los cambios.

### B. Lo que TÚ tenés que hacer manualmente (navegador — Claude Code no puede)

Son acciones de dashboard/portal que una herramienta de terminal no ejecuta:

1. **Cloudflare:** crear cuenta → **Add site** `calculatuhipotecario.cl` → plan Free → copiar los **2 nameservers** que entrega.
2. **NIC Chile** (`nic.cl`): reemplazar los nameservers del dominio por los de Cloudflare → guardar. (Propagación: minutos a 24h. NIC Chile no tiene editor de DNS, por eso se delega el dominio completo.)
3. **Cloudflare** → **Workers & Pages** → **Create application** → pestaña **Pages** → **Connect to Git** → autorizar GitHub → elegir el repo → **build settings:**
   - Framework preset: `None`
   - Build command: `exit 0`
   - Build output directory: `/`
   - **Save and Deploy.** Queda live en `<proyecto>.pages.dev`.
4. **Cloudflare** → proyecto → **Custom domains** → agregar `calculatuhipotecario.cl` (SSL automático, apex vía CNAME flattening).

Después de hacer B una sola vez, el loop queda automático (editar → push → deploy).

### Alternativa opcional para Cloudflare

Si preferís que Claude Code también maneje el deploy de Cloudflare, puede correr `wrangler pages deploy . --project-name=calculatuhipotecario` tras un `wrangler login` único. **Trade-off:** esto da deploys disparados por CLI, no por push a Git, y **igual NO cubre** el cambio de nameservers en NIC Chile (siempre manual).

---

## Spec de la mejora A3 — UF en vivo

Fuente: `https://mindicador.cl/api/uf` (datos del Banco Central, gratis, CORS habilitado, JSON). Último valor en `serie[0].valor`.

```js
async function cargarUF() {
  const FALLBACK = 40793; // respaldo si la API falla — la página nunca se rompe
  try {
    const r = await fetch('https://mindicador.cl/api/uf');
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const d = await r.json();
    const valor = d.serie?.[0]?.valor;
    if (!valor) throw new Error('sin dato');
    return { valor, fecha: d.serie[0].fecha, vivo: true };
  } catch {
    return { valor: FALLBACK, fecha: null, vivo: false };
  }
}
```

Al cargar: escribir `valor` en el input `#ufValue`, marcar estado "UF en vivo" vs "respaldo", y recalcular. No romper la página si el fetch falla.

---

## Disclaimer del sitio (ya presente en index.html, mantener)

Herramienta referencial, no es asesoría financiera ni preaprobación. Las condiciones reales dependen del banco, historial (REDEC), antigüedad laboral y tasación.
