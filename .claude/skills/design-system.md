# Design System - Beyond

## Philosophy

**Refined and premium** while keeping a serious, trustworthy tone. Think private banking, not startup.

### Design Principles

- **Quiet confidence**: Navy and gold convey trust and heritage
- **Depth through subtlety**: Soft shadows, fine borders, layered surfaces
- **Intentional whitespace**: Generous padding, content breathes
- **Micro-interactions**: Smooth transitions that feel expensive

---

## Brand Color Palette

### Primary Colors (CSS Variables in `globals.css`)

```css
:root {
  /* Beyond Brand Colors */
  --navy-deep: 217 69% 30%; /* #173C7F - textes, headers */
  --navy-light: 215 56% 46%; /* #3262B7 - liens, textes secondaires */
  --gold-heritage: 43 86% 38%; /* #B8860B - accents, CTAs, bordures actives */
  --gold-soft: 43 64% 56%; /* #D4A84B - hovers, highlights */
  --cream: 43 50% 98%; /* #FDFBF7 - background principal */
  --warm-gray: 40 22% 96%; /* #F7F5F2 - cards, sections alternees */
  --slate: 215 16% 47%; /* #64748b - texte body */
}
```

### Semantic Mappings

| Semantic Token       | Maps To       | Usage                 |
| -------------------- | ------------- | --------------------- |
| `primary`            | navy-deep     | Main text, headers    |
| `primary-foreground` | cream         | Text on primary bg    |
| `secondary`          | warm-gray     | Secondary backgrounds |
| `accent`             | gold-heritage | CTAs, active states   |
| `background`         | cream         | Main background       |
| `foreground`         | navy-deep     | Body text             |
| `muted`              | warm-gray     | Muted backgrounds     |
| `muted-foreground`   | slate         | Muted text            |

### Tailwind Usage

```tsx
// Primary button (gold CTA)
<button className="bg-gold-heritage text-cream hover:bg-gold-soft">

// Links
<a className="text-navy-light hover:text-gold-heritage">

// Cards
<div className="bg-warm-gray border-border/50">

// Headings
<h1 className="text-navy-deep font-serif-brand">
```

---

## Typography

### Fonts

- **Serif (headings)**: Georgia, 'Times New Roman', serif - via `font-serif-brand`
- **Sans (body)**: Inter via `--font-sans`
- **Display**: Fraunces via `--font-display` (alternate for special headings)

### Font Sizes (Tailwind)

```typescript
fontSize: {
  'display-lg': ['3.5rem', { lineHeight: '1.1', letterSpacing: '-0.02em' }],
  'display': ['2.5rem', { lineHeight: '1.15', letterSpacing: '-0.02em' }],
  'display-sm': ['1.875rem', { lineHeight: '1.2', letterSpacing: '-0.01em' }],
}
```

### Usage

```tsx
<h1 className="font-serif-brand text-display-lg text-navy-deep">Hero Title</h1>
<h2 className="font-serif-brand text-display-sm text-navy-deep">Section Title</h2>
<p className="text-slate text-lg leading-relaxed">Body text</p>
```

---

## Logo

The `Logo` component in `components/ui/Logo.tsx` supports three variants:

```tsx
import { Logo } from '@/components/ui';

<Logo variant="full" />     // Full logo with symbol + text (default)
<Logo variant="symbol" />   // Symbol only (for mobile/favicon)
<Logo variant="text" />     // Text only with underline
```

---

## Shadows

```typescript
boxShadow: {
  'soft': '0 2px 8px -2px rgba(0,0,0,0.05), 0 4px 16px -4px rgba(0,0,0,0.05)',
  'soft-md': '0 4px 12px -4px rgba(0,0,0,0.08), 0 8px 24px -8px rgba(0,0,0,0.06)',
  'soft-lg': '0 8px 24px -8px rgba(0,0,0,0.1), 0 16px 48px -16px rgba(0,0,0,0.08)',
  'inner-soft': 'inset 0 2px 4px 0 rgba(0,0,0,0.02)',
}
```

---

## Glassmorphism

Our premium aesthetic uses glassmorphism for cards, modals, and elevated UI elements. This creates depth while maintaining the refined, trustworthy feel.

### Core Principles

1. **Layered structure**: Gradient background -> Glass overlay -> Border -> Content
2. **Subtle colors**: Use very low opacity color tints (40-60%)
3. **Soft blur**: `backdrop-blur-[2px]` for subtle, `backdrop-blur-md` for modals
4. **White overlay**: Semi-transparent white creates the frosted effect
5. **Colored glow**: Subtle colored shadows add depth without harshness

### Glass Card Pattern

```tsx
<div className="relative rounded-2xl overflow-hidden shadow-lg">
  {/* 1. Gradient background with color tint */}
  <div className="absolute inset-0 bg-gradient-to-br from-amber-50/40 via-orange-50/20 to-white/10" />

  {/* 2. Glass overlay */}
  <div className="absolute inset-0 bg-white/30 backdrop-blur-[2px]" />

  {/* 3. Subtle border */}
  <div className="absolute inset-0 rounded-2xl border border-white/40" />

  {/* 4. Content */}
  <div className="relative p-6">{children}</div>
</div>
```

### Glass Accent Colors

| Accent  | Gradient                                        | Border                  | Glow                    |
| ------- | ----------------------------------------------- | ----------------------- | ----------------------- |
| neutral | `from-slate-50/40 via-gray-50/20 to-white/10`   | `border-white/40`       | none                    |
| gold    | `from-amber-50/40 via-orange-50/20 to-white/10` | `border-amber-100/20`   | `shadow-amber-100/10`   |
| emerald | `from-emerald-50/40 via-teal-50/20 to-white/10` | `border-emerald-100/20` | `shadow-emerald-100/10` |
| amber   | `from-amber-50/50 via-yellow-50/25 to-white/10` | `border-amber-200/30`   | `shadow-amber-100/15`   |

### Using BentoCard with Glass

```tsx
import { BentoCard } from '@/components/features/dashboard';

// Default glass (neutral)
<BentoCard>Content</BentoCard>

// Gold accent (keepsakes, CTAs)
<BentoCard glassAccent="gold">Content</BentoCard>

// Emerald accent (success states)
<BentoCard glassAccent="emerald">Content</BentoCard>

// Solid variant (no glass, original styling)
<BentoCard variant="solid">Content</BentoCard>
```

### Glass Modal Pattern

```tsx
<div className="fixed inset-0 bg-navy-deep/30 backdrop-blur-sm flex items-center justify-center">
  <div className="relative rounded-2xl overflow-hidden shadow-xl max-w-md">
    {/* Glass background - stronger blur for modals */}
    <div className="absolute inset-0 bg-gradient-to-br from-slate-50/80 via-gray-50/60 to-white/40" />
    <div className="absolute inset-0 bg-white/50 backdrop-blur-md" />
    <div className="absolute inset-0 rounded-2xl border border-white/60" />

    {/* Content */}
    <div className="relative p-6">{children}</div>
  </div>
</div>
```

### Glass Icon Container

```tsx
<div className="w-10 h-10 rounded-full bg-gold-heritage/10 backdrop-blur-sm border border-gold-heritage/20 flex items-center justify-center">
  <Icon className="w-5 h-5 text-gold-heritage" />
</div>
```

### When to Use Glassmorphism

**Use glass for:**

- Dashboard cards (BentoCard)
- Keepsake cards
- Modal dialogs
- Floating panels
- Status badges
- Icon containers

**Keep solid for:**

- Form inputs (need clear boundaries)
- Primary buttons (need visual weight)
- Navigation (already has blur)
- Footer (grounding element)

---

## Border Radius

```typescript
borderRadius: {
  xl: '1rem',
  '2xl': '1.25rem',
}
```

---

## Animations

```typescript
animation: {
  'fade-in': 'fadeIn 0.5s ease-out',
  'slide-up': 'slideUp 0.5s ease-out',
}
```

All interactive elements use `transition-all duration-200 ease-out`.

---

## Component Patterns

### Cards

```tsx
<div className="bg-warm-gray rounded-2xl border border-border/50 shadow-soft p-8 transition-shadow duration-200 ease-out hover:shadow-soft-md">
  {/* content */}
</div>
```

### Primary Button (Gold CTA)

```tsx
<button className="bg-gold-heritage text-cream hover:bg-gold-soft rounded-xl px-8 py-4 font-medium shadow-soft transition-all duration-200 ease-out hover:shadow-soft-md">
  Get Started
</button>
```

### Secondary Button

```tsx
<button className="border border-border/60 text-foreground rounded-xl px-8 py-4 font-medium transition-colors duration-200 ease-out hover:bg-muted/50">
  Learn More
</button>
```

### Links

```tsx
<a className="text-navy-light hover:text-gold-heritage transition-colors duration-200 ease-out">
  Click here
</a>
```

### Inputs

```tsx
<input className="rounded-xl border-border/60 bg-background px-4 py-3 shadow-inner-soft focus:border-gold-heritage focus:ring-gold-heritage/50 transition-colors" />
```

### Section Spacing

```tsx
<section className="py-24 px-6">
  <div className="max-w-4xl mx-auto space-y-16">{/* content */}</div>
</section>
```

### Header/Navbar

```tsx
<header className="border-b border-warm-gray bg-cream/80 backdrop-blur-sm sticky top-0 z-50">
```

---

## Selection Styling

```css
::selection {
  @apply bg-gold-heritage/20 text-foreground;
}
```

---

## Focus Rings

```css
focus:ring-gold-heritage/50
focus:border-gold-heritage
```

---

## Key Rules

1. **Headings**: Always use `font-serif-brand` for h1, h2
2. **CTAs**: Use `bg-gold-heritage` with `text-cream`
3. **Links**: Use `text-navy-light hover:text-gold-heritage`
4. **Cards**: Use glassmorphism (BentoCard) for dashboard cards, keepsakes
5. **Modals**: Use glass pattern with `backdrop-blur-md` and layered structure
6. **Transitions**: Always `duration-200 ease-out`
7. **Borders**: Use `border-white/40` for glass, `border-border/50` for solid
8. **Shadows**: Replace harsh shadows with `shadow-soft` variants
9. **Spacing**: Generous - sections get `py-24`, cards get `p-6`
10. **Focus states**: Use `ring-gold-heritage/50`
11. **Glass accents**: Use gold for keepsakes/CTAs, emerald for success, amber for warnings
