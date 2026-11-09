---
name: dust-react-hook-form-writer
dust_skill_id: skl_KA5YQGJEHA0r
availability: workspace_users
---

# dust-react-hook-form-writer

**Description:** Write and refactor React forms using react-hook-form with Zod validation. Use when creating new form components, converting existing forms to react-hook-form, or implementing form validation patterns.

## Instructions

# React Hook Form Writer
This skill helps you write new forms and refactor existing forms to use react-hook-form following project best practices.

## When to Use
- Creating new form components from scratch
- Converting existing forms to react-hook-form
- Adding validation to forms
- Implementing complex form patterns (nested forms, field arrays, multi-step)

## Core Principles

### 1. Always Use Zod for Validation
```typescript
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";

const formSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email address"),
  age: z.number().min(18, "Must be at least 18"),
});

type FormValues = z.infer<typeof formSchema>;

const form = useForm<FormValues>({
  resolver: zodResolver(formSchema),
  defaultValues: { name: "", email: "", age: 18 },
});
```

### 2. Prefer useController Over Controller
```typescript
// Good: useController
function TextField({ name, control, label }: TextFieldProps) {
  const { field, fieldState } = useController({ name, control });
  return (
    <div>
      <label>{label}</label>
      <input {...field} />
      {fieldState.error && <span>{fieldState.error.message}</span>}
    </div>
  );
}
// Avoid: Controller component (less composable)
<Controller name="name" control={control} render={({ field }) => <input {...field} />} />
```

### 3. Uncontrolled by Default
```typescript
// Good: Uncontrolled with register
<input {...register("name")} />
// Only use Controller/useController for third-party controlled components
// (e.g., shadcn Select, custom date pickers, rich text editors)
```

### 4. Use field.onChange for User Interactions, setValue for Programmatic Updates
```typescript
// Good: field.onChange for user interactions
const { field } = useController({ name: "status", control });
<Select onValueChange={field.onChange} value={field.value}>
  {options.map((opt) => <SelectItem key={opt.value} value={opt.value}>{opt.label}</SelectItem>)}
</Select>
// Bad: setValue for user interactions (breaks controller lifecycle)
<Select onValueChange={(v) => setValue("status", v)} value={watch("status")}>
```
Use `setValue` (from `useFormContext`) for programmatic updates in effects:
```typescript
const { setValue } = useFormContext();
const { field } = useController({ name: "status" });
useEffect(() => {
  if (externalData) setValue("status", externalData.defaultStatus); // programmatic
}, [externalData, setValue]);
const handleUserSelect = (value: string) => field.onChange(value); // user interaction
```
**CRITICAL: Never use field.onChange inside useEffect dependencies.**
`useController` returns new field objects on every render. Including them in `useEffect` dependencies while also calling `field.onChange()` inside the effect causes infinite loops:
```typescript
// BAD: Infinite loop - field objects change every render
const { field } = useController({ name: "status" });
useEffect(() => { field.onChange(defaultValue); }, [field, defaultValue]);

// GOOD: Use setValue (stable) for programmatic updates in effects
const { setValue } = useFormContext();
const { field } = useController({ name: "status" });
useEffect(() => { setValue("status", defaultValue); }, [defaultValue, setValue]);
// User interactions still use field.onChange
const handleSelect = (value: string) => { field.onChange(value); };
```
**Summary:**
- `field.onChange` → user interaction handlers (onClick, onSelect, etc.)
- `setValue` → programmatic updates in useEffect or callbacks based on external data

### 5. Always Provide Default Values
```typescript
// Good: All fields have defaults
const form = useForm<FormValues>({
  resolver: zodResolver(formSchema),
  defaultValues: { name: "", email: "", items: [], settings: { notifications: true, theme: "light" } },
});
// Bad: Missing defaultValues causes controlled/uncontrolled warnings
const form = useForm<FormValues>({ resolver: zodResolver(formSchema) });
```

### 6. Watch Specific Fields Only
```typescript
// Good: Watch specific fields
const selectedType = watch("type");
const [name, email] = watch(["name", "email"]);
// Bad: Watches everything, causes unnecessary re-renders
const allValues = watch();
```

### 7. Use Dot Notation for Nested Fields
```typescript
const schema = z.object({
  user: z.object({ profile: z.object({ firstName: z.string(), lastName: z.string() }) }),
});
<input {...register("user.profile.firstName")} />
```

### 8. Use useFieldArray for Dynamic Lists
```typescript
const { fields, append, remove } = useFieldArray({ control, name: "items" });
return (
  <div>
    {fields.map((field, index) => (
      <div key={field.id}>
        <input {...register(`items.${index}.name`)} />
        <button type="button" onClick={() => remove(index)}>Remove</button>
      </div>
    ))}
    <button type="button" onClick={() => append({ name: "" })}>Add Item</button>
  </div>
);
```

### 9. Proper Form Submission
```typescript
const onSubmit = async (data: FormValues) => {
  try {
    await submitToApi(data);
  } catch (error) {
    form.setError("root", { message: "Submission failed" });
  }
};
<form onSubmit={form.handleSubmit(onSubmit)}>
  {form.formState.errors.root && <div className="error">{form.formState.errors.root.message}</div>}
  <button type="submit" disabled={form.formState.isSubmitting}>Submit</button>
</form>
```

### 10. Reset Forms Correctly
```typescript
// Good: Reset with new values
form.reset({ name: "New Name", email: "new@email.com" });
// Good: Reset to default values
form.reset();
// Bad: Manual field clearing
setValue("name", ""); setValue("email", "");
```

### 11. Sub-form Validation with trigger()
```typescript
const isStepValid = await form.trigger(["name", "email"]);
if (isStepValid) goToNextStep();
```

### 12. Error Display Pattern
```typescript
const { formState: { errors } } = form;
<input {...register("email")} />
{errors.email && <span className="text-red-500">{errors.email.message}</span>}
```

## Refactoring Checklist
When refactoring existing forms to react-hook-form:
- [ ] Define Zod schema matching existing validation
- [ ] Set up useForm with zodResolver and defaultValues
- [ ] Replace controlled inputs with register() where possible
- [ ] Use useController for third-party controlled components
- [ ] Replace manual state management with form state
- [ ] Convert submit handlers to use handleSubmit
- [ ] Update error display to use formState.errors
- [ ] Replace manual arrays with useFieldArray
- [ ] Remove unnecessary useState for form values
