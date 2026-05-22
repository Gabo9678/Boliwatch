# Notas de desarrollo — Boliwatch

> Tracker de inversiones personal adaptado al mercado venezolano:
> múltiples tasas (BCV, paralelo, Binance P2P), múltiples activos
> (FIAT y cripto), histórico de movimientos.

## Stack

- **Next.js 16.2.6** con App Router y Turbopack
- **React 19.2.4**
- **TypeScript 5**
- **Tailwind CSS 4** 
- **Prisma + PostgreSQL (Neon)** — pendiente de configurar
- **Auth.js (NextAuth v5)** — pendiente
- **Recharts** para gráficos — pendiente
- **TanStack Query** para fetching — pendiente
- **Zod** para validación — pendiente
- **pnpm** como package manager

## Decisiones tomadas

- **Modelo de doble entrada (double-entry):** una `Transaction` tiene N
  `TransactionEntry`. Permite representar compras, depósitos, conversiones,
  transferencias y retiros con el mismo modelo. Cada entry tiene un `amount`
  con signo: positivo = entra, negativo = sale.
- **Una sola tabla `Asset`** para FIAT y cripto. Comparten estructura,
  diferenciados por un campo `type`. Evita tener `if (esCripto)` repartido
  por el código.
- **`Decimal` en vez de `Float`** para todo lo monetario. Los floats tienen
  errores de precisión inaceptables en finanzas (0.1 + 0.2 ≠ 0.3 en JS).
- **Histórico de tasas (`ExchangeRate`):** cada transacción captura la
  tasa del día en `rateUsed`. NO se recalcula con la tasa actual —
  inmutabilidad de hechos pasados.
- **Registro manual de movimientos en el MVP.** Descartada la integración
  con Binance API por complejidad de seguridad (manejar API keys de
  usuarios). Queda para v2.
- **Solo cuentas digitales en el MVP.** Sin efectivo, sin cuentas bancarias
  conectadas. Solo registro manual.

## Modelo de datos (resumen)

- `User` → tiene muchas `Account` y `Transaction`
- `Account` → "Mi Binance", "Mercantil Bs", "Zelle"... cada una con un `AccountType`
- `Asset` → USDT, VES, BTC, USD, EUR... cada uno con `AssetType` (FIAT o CRYPTO)
- `Transaction` → evento del mundo real ("compré 100 USDT")
- `TransactionEntry` → pierna individual del movimiento (entra o sale algo de algún lado)
- `ExchangeRate` → histórico automático de tasas (BCV, PARALELO, BINANCE_P2P, EURO_BCV)

## MVP — features

1. Auth (Google login)
2. CRUD de cuentas del usuario
3. Registro de movimientos (compra, venta, depósito, retiro, transferencia, conversión)
4. Servicio que actualiza tasas automáticamente
5. Dashboard: patrimonio total convertido a las 3 tasas + distribución por activo/cuenta
6. Histórico filtrable

## Dudas pendientes

- ¿De dónde sacar las tasas de forma confiable? Investigar APIs públicas
  o scrapers. Opciones: pyDolarVenezuela, exchangemonitor, dolarapi.
- ¿Cron job desde Vercel o un servicio externo para refrescar tasas?

## Comandos útiles

\`\`\`bash
pnpm dev          # arranca el servidor de desarrollo con Turbopack
pnpm build        # build de producción
pnpm lint         # linter
pnpm approve-builds  # aprueba scripts