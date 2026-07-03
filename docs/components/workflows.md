# Workflows and providers

This package includes a few higher-level modules that solve more than a single UI primitive:

- stacked modal orchestration
- multi-step wizards
- webcam capture and upload flows
- time input controls
- cloud storage provider wiring

## Provider checklist

| Feature | Required setup |
| --- | --- |
| `@medram/ui/modal` | Wrap the app with `StackedModalsProvider` |
| `@medram/ui/webcam` | Wrap the app with `CloudStorageProvider` |
| `WebcamImageUploadModal` | `StackedModalsProvider` + `CloudStorageProvider` + Formik |
| `@medram/ui/time-picker` | No provider required |
| `@medram/ui/wizard` | No provider required for the high-level `Wizard` component |

## Wizard module

Use `@medram/ui/wizard` when you want step orchestration without rebuilding navigation, progress, and async transitions.

### Best exports

| Export | Use it for |
| --- | --- |
| `Wizard` | High-level drop-in multi-step flow. |
| `BaseWizard` | Low-level composition API for custom layouts. |
| `useWizardContext` | Advanced step-aware custom components. |
| `WizardStep`, `WizardController`, `ProgressBarRenderProps` | Type-safe wizard configuration. |

### Example: account setup wizard

```tsx
import { Wizard, type WizardStep } from "@medram/ui/wizard"

const steps: WizardStep[] = [
  { id: "profile", title: "Profile", content: <ProfileStep /> },
  { id: "permissions", title: "Permissions", content: <PermissionsStep /> },
  { id: "review", title: "Review", content: <ReviewStep /> },
]

export function AccountSetupWizard() {
  return (
    <Wizard
      steps={steps}
      showProgressBar
      showNavigation
      onFinish={async () => saveAccount()}
    />
  )
}
```

## Modal module

Use `@medram/ui/modal` when one dialog can lead to another and you want stack-aware behavior instead of manual local state in every screen.

### Best exports

| Export | Use it for |
| --- | --- |
| `StackedModalsProvider` | App-level modal stack state. |
| `useModalContext` | Opening and closing stack-aware modals from custom components. |
| `TriggerModal` | Consistent button-style entry into modal flows. |
| `ModalStackedBox` | Low-level stacked modal box. |
| `DialogStack*` | Custom dialog layouts built on the same modal stack primitives. |

### Example: provider plus trigger

```tsx
import { StackedModalsProvider, TriggerModal, useModalContext } from "@medram/ui/modal"

function InviteUserButton() {
  const { open } = useModalContext()

  return (
    <TriggerModal
      onClick={() =>
        open({
          title: "Invite teammate",
          description: "Send access to a new user.",
          modal: ({ close }) => <InviteForm onSuccess={close} />,
        })
      }
    >
      Invite teammate
    </TriggerModal>
  )
}

export function AppProviders({ children }: { children: React.ReactNode }) {
  return <StackedModalsProvider>{children}</StackedModalsProvider>
}
```

## Webcam module

Use `@medram/ui/webcam` when users should either upload an image or capture one from the camera.

### Best exports

| Export | Use it for |
| --- | --- |
| `WebcamImageUploader` | Full upload-or-capture flow with save callback. |
| `WebcamCapture` | Low-level capture surface for custom flows. |
| `ImageUploadCard` | Presentation component for file selection, preview, and progress. |
| `WebcamImageUploadModal` | Formik-integrated modal-based image selection flow. |

### Example: non-modal upload-or-capture flow

```tsx
import { CloudStorageProvider } from "@medram/ui/cloud-storage"
import { WebcamImageUploader } from "@medram/ui/webcam"

export function ProfileImageStep() {
  return (
    <CloudStorageProvider value={cloudStorageValue}>
      <WebcamImageUploader
        label="Choose a profile image"
        webcamType="portrait"
        onSave={(attachment) => console.log("saved", attachment?.id)}
      />
    </CloudStorageProvider>
  )
}
```

### Example: modal-based Formik image field

```tsx
import { Form, Formik } from "formik"
import { CloudStorageProvider } from "@medram/ui/cloud-storage"
import { StackedModalsProvider } from "@medram/ui/modal"
import { WebcamImageUploadModal } from "@medram/ui/webcam"

export function ProfileImageField() {
  return (
    <StackedModalsProvider>
      <CloudStorageProvider value={cloudStorageValue}>
        <Formik initialValues={{ avatarLink: null as string | null }} onSubmit={saveProfile}>
          <Form>
            <WebcamImageUploadModal
              name="avatarLink"
              label="Profile image"
              aspect="1:1"
              webcamType="portrait"
            />
          </Form>
        </Formik>
      </CloudStorageProvider>
    </StackedModalsProvider>
  )
}
```

`WebcamImageUploadModal` writes the saved attachment **link** into the named Formik field, so a `string | null` value is the right starting shape. This differs from `UploadField`, which typically stores attachment IDs.

## Time picker module

Use `@medram/ui/time-picker` when you need a standalone time input without a full date picker.

| Export | Use it for |
| --- | --- |
| `TimePicker` | Complete hour/minute/second input with 12h or 24h mode. |
| `TimePickerInput` | Low-level input building block for custom layouts. |

### Example: time-only control

```tsx
import { useState } from "react"
import { TimePicker } from "@medram/ui/time-picker"

export function QuietHoursInput() {
  const [value, setValue] = useState(new Date())

  return <TimePicker date={value} onChange={setValue} hourCycle={12} granularity="minute" />
}
```

## Cloud storage provider

Use `@medram/ui/cloud-storage` to bridge your app's upload APIs into the package.

| Export | Use it for |
| --- | --- |
| `CloudStorageProvider` | Inject upload, fetch, and delete behavior into the component tree. |
| `useCloudStorageContext` | Read the same contract from custom app code. |
| `CloudStorageContextValue` | Type the provider value correctly. |
| `UploadFileOptions` | Type upload options such as progress reporting and server-side filename selection. |

### Example: provider contract

```tsx
import {
  CloudStorageProvider,
  type CloudStorageContextValue,
} from "@medram/ui/cloud-storage"

const cloudStorageValue: CloudStorageContextValue = {
  uploadFile: async (file, options) => uploadViaApi(file, options),
  fetchAttachment: async (id) => fetchAttachment(id),
  deleteAttachment: async (id) => deleteAttachment(id),
  onError: (error) => reportError(error),
}

export function AppProviders({ children }: { children: React.ReactNode }) {
  return <CloudStorageProvider value={cloudStorageValue}>{children}</CloudStorageProvider>
}
```

For the full contract shape and host-app integration notes, see the [cloud storage reference](/reference/cloud-storage).
