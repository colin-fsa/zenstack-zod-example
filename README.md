# ZenStack Zod Plugin Issue Reproduction

This repository is a minimal reproduction example (MRE) demonstrating an issue with ZenStack's Zod schema generation.

## The Issue

When using ZenStack with models that have validation rules (like regex validation), there's an inconsistency in how Zod schemas are generated depending on whether the Zod plugin is defined in the schema.zmodel file:

1. **Without the Zod plugin defined**: 
   - Zod schemas are saved to `@zenstackhq/runtime/zod/models` (in node_modules)
   - These schemas **do not include** the validation rules specified in the model

2. **With the Zod plugin defined** (as shown in our schema.zmodel):
   ```
   plugin zod {
     provider = '@core/zod'
     output = './zod'
   }
   ```
   - Schemas are still written to `@zenstackhq/runtime/zod/models` 
   - But the **correct schema with regex validation** is also written to the specified output path

## Example

In our example User model with zipCode regex validation:

```zmodel
model User {
  id      String  @id @default(cuid())
  zipCode String? @regex('^[0-9a-zA-Z]{4,16}$')
}
```

The Zod schema in node_modules does not include the regex validation:
```typescript
// From node_modules/.zenstack/zod/models/User.schema.d.ts
export declare const UserScalarSchema: z.ZodObject<{
    id: z.ZodString;
    zipCode: z.ZodOptional<z.ZodNullable<z.ZodString>>;
}, "strict", z.ZodTypeAny, {
    id: string;
    zipCode?: string | null | undefined;
}, {
    id: string;
    zipCode?: string | null | undefined;
}>;
```

While the schema generated in our project's ./zod directory includes the validation:
```typescript
// From ./zod/models/User.schema.ts
const baseSchema = z.object({
    id: z.string(),
    zipCode: z.string().regex(new RegExp("^[0-9a-zA-Z]{4,16}$")).nullish(),
}).strict();
```
