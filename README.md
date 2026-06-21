# Simula tu hipotecario

**Simulador de crédito hipotecario en Chile**: dividendo mensual, **renta líquida mínima requerida**, monto del crédito y costo total, según el valor de una propiedad.

- **Dominio principal:** [simulatuhipotecario.cl](https://simulatuhipotecario.cl)
- **Alias con redirect 301:** `calculatuhipotecario.cl` → `simulatuhipotecario.cl` (el repositorio conserva el nombre histórico).

Sitio 100% estático: un solo archivo `index.html` con CSS/JS inline, sin backend ni base de datos.

## Características

- Renta líquida mínima requerida según valor de propiedad, pie, plazo y tasa.
- Doble criterio de evaluación: dividendo/renta (25% estricto / 30% flexible) y carga financiera total (45%).
- Desglose de dividendo, caja necesaria al firmar y costo total del crédito (amortización francesa).
- **UF en vivo** desde [mindicador.cl](https://mindicador.cl/api/uf) (Banco Central), con valor de respaldo si la API falla e indicador de estado.
- Tasas de referencia del mercado e inputs editables.

## Desarrollo

No requiere build. Abrí `index.html` en el navegador.

## Deploy

Sitio estático servido por Cloudflare Pages, conectado a este repo. Cada push a `main` redespliega automáticamente.

- Framework preset: `None`
- Build command: `exit 0`
- Build output directory: `/`

## Disclaimer

Herramienta referencial, no es asesoría financiera ni preaprobación. Las condiciones reales dependen del banco, historial (REDEC), antigüedad laboral y tasación.
