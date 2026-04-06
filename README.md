# Modern React Stack Blueprint

> Full library audit & migration roadmap — replacing ~4,200 lines of manually-written code with battle-tested libraries.

Built by [Aman Soni](https://github.com/amansoni1605) · [Portfolio](https://github.com/amansoni1605/portfolio)

## What This Is

A comprehensive architectural audit of a production Next.js 15 financial platform (`advisor-web`).  
Every manually-built system was identified and mapped to the correct battle-tested library.

| Metric | Count |
|---|---|
| Lines of custom code replaced | ~4,200 |
| Libraries added | 23 |
| Libraries removed | 8 |
| Libraries kept | 12 |

## Categories Covered

| # | Category | Priority | Key Replacements |
|---|---|---|---|
| 1 | [Data Fetching & Caching](docs/01-data-fetching.md) | P1 — Day One | 810-line fetchUtils → Axios + TanStack Query |
| 2 | [Forms & Validation](docs/02-forms.md) | P1 — Day One | 18KB InputElement → React Hook Form + Zod |
| 3 | [UI Components & Primitives](docs/03-ui-components.md) | P2 — Sprint 1 | Custom Modal/Toast/Tooltip → Radix UI + Sonner |
| 4 | [Authentication](docs/04-auth.md) | P1 — Day One | Custom cookie logic → Auth.js v5 |
| 5 | [State Management](docs/05-state.md) | P2 — Sprint 1 | 24 Redux slices → Zustand + React Query |
| 6 | [CMS & Content](docs/06-cms.md) | P1 — Foundation | 640 JSON files → Payload CMS v3 |
| 7 | [Tables & Data Grids](docs/07-tables.md) | P2 — Sprint 1 | Custom tables → TanStack Table + Virtual |
| 8 | [Charts & Visualization](docs/08-charts.md) | P3 — Feature Sprint | Highcharts (keep) + Recharts (add) |
| 9 | [Utilities & DX](docs/09-utilities.md) | P2 — Sprint 1 | t3-env, date-fns, Sentry, Vitest, Playwright |

## Quick Summary: Old vs. New

| Concern | Old | New | Win |
|---|---|---|---|
| API calls | `fetchUtils.ts` — 810 lines | Axios + React Query — 60 lines | −750 lines |
| Server state | 12 Redux slices | TanStack Query | −12 slices |
| UI state | Redux + RTK — 50KB+ | Zustand — 2KB | −48KB |
| Forms | `InputElement.tsx` — 18KB | React Hook Form + Zod | −18KB |
| Toast | Custom `Alerts.tsx` + Redux slice | Sonner — 1.2KB | −200 lines, −1 slice |
| Modal | Custom with broken a11y | Radix UI Dialog (already installed) | 0 installs needed |
| Auth | Custom cookie logic, manual refresh | Auth.js v5 | −500 lines |
| CMS schema | 640 JSON files | Payload CMS v3 — TypeScript blocks | −470 files |
| Testing | Zero coverage | Vitest + Playwright | Regressions caught pre-prod |
| Error monitoring | None | Sentry | Proactive detection |

## Install Script

```bash
# P1: Day One (foundation)
pnpm add @tanstack/react-query @tanstack/react-query-devtools
pnpm add axios ky
pnpm add next-auth@beta
pnpm add payload drizzle-orm drizzle-kit
pnpm add react-hook-form @hookform/resolvers zod
pnpm add zustand immer
pnpm add @t3-oss/env-nextjs

# P2: Sprint 1 (UI & DX)
pnpm add sonner framer-motion lucide-react next-themes
pnpm add react-error-boundary react-day-picker
pnpm add embla-carousel-react @tanstack/react-virtual
pnpm add date-fns numeral @sentry/nextjs

# P3: Pre-launch
pnpm add recharts
pnpm add -D @playwright/test vitest @testing-library/react
pnpm add -D @next/bundle-analyzer storybook

# Remove
pnpm remove react-slick react-tooltip flatpickr jsencrypt
pnpm remove @strapi/blocks-react-renderer
pnpm remove @reduxjs/toolkit react-redux
```

## Key Code Patterns

### TanStack Query replacing Redux server-cache slices

```ts
// BEFORE: 12 Redux slices (~2,000 lines)
dispatch(fetchPortfolio())
const { data, loading } = useSelector(selectPortfolio)

// AFTER: React Query (15 lines, zero stale data)
export const PORTFOLIO_KEYS = {
  all:    () => ['portfolio'] as const,
  detail: (id: string) => [...PORTFOLIO_KEYS.all(), id] as const,
}

export const usePortfolio = (id: string) =>
  useQuery({
    queryKey: PORTFOLIO_KEYS.detail(id),
    queryFn:  () => api.get(`/portfolio/${id}`),
    staleTime: 5 * 60 * 1000,
  })
```

### Zustand replacing Redux UI slices

```ts
// BEFORE: Redux modal slice (~50 lines boilerplate)
dispatch(openModal({ id: 'confirm', props: { message } }))

// AFTER: Zustand (entire store in 15 lines)
export const useModalStore = create<ModalStore>()(devtools(set => ({
  id: null, props: {},
  open: (id, props = {}) => set({ id, props }),
  close: () => set({ id: null, props: {} }),
})))

const { open } = useModalStore()
open('confirm', { message })
```

### Axios interceptors replacing fetchUtils auth logic

```ts
// BEFORE: Auth refresh scattered across 810-line fetchUtils.ts

// AFTER: Single interceptor
axiosClient.interceptors.response.use(
  (res) => res,
  async (err) => {
    if (err.response?.status === 401 && !err.config._retry) {
      err.config._retry = true
      await refreshToken()
      return axiosClient(err.config)
    }
    return Promise.reject(err)
  }
)
```

---

*Companion to the advisor-web migration project.*
