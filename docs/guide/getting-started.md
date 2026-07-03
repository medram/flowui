# Getting started

## Install

```bash
npm install @medram/ui
```

If you plan to use the Formik-ready field layer, install `formik` in the host app too.
## Add the Tailwind preset

```ts
import preset from "@medram/ui/tailwind"

const config = {
  presets: [preset],
}

export default config
```

Host apps also need Tailwind content scanning for the packaged build output:

```ts
content: ["./node_modules/@medram/ui/dist/**/*.{js,mjs}"]
```

## Load design tokens

Import the shared stylesheet once near the top of your app shell:

```ts
import "@medram/ui/styles.css"
```

Skip this only if the host app already defines the same CSS variables.
## Import components

Use the root entrypoint for shared widgets and helpers:

```tsx
import { SubmitButton, Tabs } from "@medram/ui"
```

Use subpaths when you want a narrower dependency boundary:

```tsx
import { InputField } from "@medram/ui/fields"
import { Wizard } from "@medram/ui/wizard"
import { Button } from "@medram/ui/primitives"
```

For form-heavy screens, the clean default is:

- field components from `@medram/ui/fields`
- shared helpers such as `SubmitButton` from `@medram/ui`

## Upload and webcam flows

Components that access attachments require `CloudStorageProvider` from `@medram/ui/cloud-storage`.

The modal webcam field is a special case:

- `WebcamImageUploader` needs `CloudStorageProvider`
- `WebcamImageUploadModal` needs `CloudStorageProvider` + `StackedModalsProvider` + Formik

```tsx
import { CloudStorageProvider } from "@medram/ui/cloud-storage"

export function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <CloudStorageProvider value={{ /* app upload implementation */ }}>
      {children}
    </CloudStorageProvider>
  )
}
```

The provider contract is documented in the [cloud storage reference](/reference/cloud-storage).
