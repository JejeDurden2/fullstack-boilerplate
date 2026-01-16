# Frontend Architecture - React

## Folder Structure

```
apps/web/src/
├── app/                        # Next.js App Router (or pages/)
│   ├── (marketing)/            # Public pages group
│   ├── (dashboard)/            # Auth-required pages group
│   ├── api/                    # API routes (if needed)
│   ├── sitemap.ts              # Dynamic sitemap
│   └── robots.ts               # Robots.txt
├── components/
│   ├── ui/                     # shadcn/ui primitives
│   ├── common/                 # Reusable business components
│   ├── forms/                  # Form components
│   └── layouts/                # Layout components
├── features/
│   └── [feature]/
│       ├── components/         # Feature-specific components
│       ├── hooks/              # Feature-specific hooks
│       ├── api/                # API calls (TanStack Query)
│       └── types/              # Feature types
├── hooks/                      # Global hooks
├── lib/
│   ├── api-client.ts           # Axios/fetch wrapper
│   ├── utils.ts                # Utility functions
│   └── validations/            # Zod schemas
├── stores/                     # Zustand stores
└── styles/
    └── globals.css
```

---

## Component Patterns

### Reusable Component Template

```typescript
// components/common/data-table.tsx
interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  isLoading?: boolean;
  onRowClick?: (row: T) => void;
  emptyMessage?: string;
}

export function DataTable<T>({
  data,
  columns,
  isLoading = false,
  onRowClick,
  emptyMessage = 'No data found',
}: DataTableProps<T>): JSX.Element {
  // Implementation
}
```

### TanStack Query Pattern

```typescript
// features/users/api/use-users.ts
export function useUsers(filter?: UserFilter) {
  return useQuery({
    queryKey: ['users', filter],
    queryFn: () => apiClient.get<User[]>('/users', { params: filter }),
    staleTime: 5 * 60 * 1000,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateUserDto) => apiClient.post('/users', data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

### Form Pattern (React Hook Form + Zod)

```typescript
const formSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
});

type FormData = z.infer<typeof formSchema>;

export function UserForm({ onSubmit }: UserFormProps) {
  const form = useForm<FormData>({
    resolver: zodResolver(formSchema),
    defaultValues: { email: '', name: '' },
  });

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        {/* Fields */}
      </form>
    </Form>
  );
}
```

---

## Key Rules

1. **Server Components by default** — only `'use client'` when necessary
2. **Create reusable, typed components** — no one-off implementations
3. **Implement mobile-first** — then scale up (sm → md → lg → xl)
4. **Components are reusable** — no hardcoded values
5. **Colocation** — tests and styles close to components
6. **Named exports over default exports**
