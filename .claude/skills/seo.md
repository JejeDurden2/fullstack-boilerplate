# SEO Implementation

## Dynamic Metadata (Next.js)

```typescript
// app/[slug]/page.tsx
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const data = await getData(params.slug);

  return {
    title: data.title,
    description: data.description,
    openGraph: {
      title: data.title,
      description: data.description,
      images: [data.image],
    },
    alternates: {
      canonical: `https://domain.com/${params.slug}`,
    },
  };
}
```

---

## Structured Data (JSON-LD)

```typescript
// components/common/json-ld.tsx
interface JsonLdProps {
  data: WithContext<Thing>;
}

export function JsonLd({ data }: JsonLdProps): JSX.Element {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  );
}

// Usage
<JsonLd data={{
  '@context': 'https://schema.org',
  '@type': 'Product',
  name: product.name,
  description: product.description,
}} />
```

---

## Sitemap Generation

```typescript
// app/sitemap.ts
export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const pages = await getPages();

  return [
    { url: 'https://domain.com', lastModified: new Date(), priority: 1 },
    ...pages.map((page) => ({
      url: `https://domain.com/${page.slug}`,
      lastModified: page.updatedAt,
      priority: 0.8,
    })),
  ];
}
```

---

## Robots.txt

```typescript
// app/robots.ts
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://domain.com';

  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/api/', '/dashboard/'],
      },
    ],
    sitemap: `${baseUrl}/sitemap.xml`,
  };
}
```

---

## Multilingual SEO

```typescript
// app/[locale]/layout.tsx
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { locale } = await params;
  const t = await getTranslations({ locale, namespace: 'meta' });

  return {
    title: t('title'),
    description: t('description'),
    alternates: {
      canonical: `https://domain.com/${locale === 'fr' ? '' : locale}`,
      languages: {
        fr: 'https://domain.com/',
        en: 'https://domain.com/en',
      },
    },
  };
}
```

---

## Key Rules

1. **Consider SEO impact** — metadata, structure, performance, Core Web Vitals
2. **Translate metadata** — titles, descriptions, OG tags
3. **Set `hreflang` tags** — for all alternate language versions
4. **Include canonical URLs** — prevent duplicate content
5. **Generate dynamic sitemaps** — include all public pages
6. **Use JSON-LD** — for structured data (Organization, Product, etc.)
