# React Hook Form

## Why React Hook Form?

With plain React, every field can be managed with its own state. For large forms this becomes repetitive. React Hook Form reduces the amount of form state you need to manage manually and is designed to minimize unnecessary rendering.

Install:

    npm install react-hook-form

## Basic example

    import { useForm } from "react-hook-form";

    function LoginForm() {
      const { register, handleSubmit, formState: { errors } } = useForm();

      function onSubmit(data) {
        console.log(data);
      }

      return (
        <form onSubmit={handleSubmit(onSubmit)}>
          <input {...register("email")} />
          <input type="password" {...register("password")} />
          <button type="submit">Login</button>
        </form>
      );
    }

Submitted data:

    { email: "tanay@example.com", password: "secret" }

## register()

`register()` connects a normal input to React Hook Form:

    <input {...register("email")} />

It tells RHF to track the input as the `email` field.

## handleSubmit()

    <form onSubmit={handleSubmit(onSubmit)}>

`handleSubmit` handles submission and calls your function with the form data after validation.

    Submit -> handleSubmit() -> Validate -> Valid? -> onSubmit / errors

## Validation

Validation rules can be supplied to `register()`:

    <input
      {...register("email", {
        required: "Email is required",
        pattern: {
          value: /^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$/,
          message: "Invalid email"
        }
      })}
    />

Password example:

    <input
      type="password"
      {...register("password", {
        required: "Password is required",
        minLength: {
          value: 8,
          message: "Password must be at least 8 characters"
        }
      })}
    />

## formState.errors

    const { formState: { errors } } = useForm();

Display an error:

    {errors.email && <p>{errors.email.message}</p>}

## defaultValues

Useful for initial values and edit forms:

    const { register } = useForm({
      defaultValues: {
        name: "Tanay",
        email: "tanay@example.com"
      }
    });

## reset()

Useful when asynchronously fetched data needs to populate an edit form:

    const { reset } = useForm();

    useEffect(() => {
      fetchUser().then(user => reset(user));
    }, [reset]);

`reset()` replaces the form values with the supplied values.

## watch()

Use `watch()` when UI needs to react to another field:

    const { register, watch } = useForm();
    const country = watch("country");

    {country === "India" && (
      <input {...register("state")} />
    )}

Don't watch everything unnecessarily; watch only what the UI actually depends on.

## setValue() and getValues()

    setValue("email", "tanay@example.com");
    const email = getValues("email");

`setValue()` changes a field programmatically; `getValues()` reads a current value.

## Controlled components and Controller

Some third-party components require controlled `value` + `onChange`, such as DatePicker, Select, Autocomplete and RichTextEditor.

Use `Controller`:

    <Controller
      name="country"
      control={control}
      render={({ field }) => (
        <CustomSelect
          value={field.value}
          onChange={field.onChange}
        />
      )}
    />

Mental model:

    Custom component -> Controller -> React Hook Form

You generally don't need `Controller` for ordinary input/select/textarea when `register()` works directly.

## Why RHF can be performant

React Hook Form is designed to avoid putting every keystroke into React component state unnecessarily. Its uncontrolled/ref-oriented approach for native inputs can reduce unnecessary React rendering, which is useful for large forms.

This does not mean RHF never causes re-renders. Errors, watched values and subscribed form state can still cause relevant components to render.

## SDE-2 Cheat Sheet

    useForm()          -> form management
    register()         -> connect normal inputs
    handleSubmit()     -> validate + submit
    formState.errors   -> validation errors
    watch()            -> observe field values
    setValue()         -> change value programmatically
    getValues()        -> read value
    reset()            -> replace/reset form values
    Controller         -> integrate controlled/third-party components

## Best Practices

- Use `register()` for normal native inputs.
- Use `Controller` when a controlled/third-party component requires it.
- Use `defaultValues` for initial form state.
- Use `reset()` when populating an edit form from fetched data.
- Don't use `watch()` for everything; watch only what the UI actually depends on.
- Keep server-side validation too; client-side validation is for UX, not security.
- For complex validation, RHF can be paired with schema libraries such as Zod/Yup.
