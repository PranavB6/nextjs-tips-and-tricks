# Component Organization

- Since we have both client and server components, the most intuitive way to organize component is to put all server components in the `app/` folder and all client components in the `components/` folder
- Personally, I have found that putting components that are specific to routes in the `app/` folder and putting all "shared" components in the `components/` folder to be easier to organize

```
# Ignoring all page.tsx and layout.tsx
app/
├─ (protected)/
│  ├─ history/
│  │  ├─ HistoryContainer.tsx
│  │  ├─ HistoryRow.tsx
│  ├─ time-entry/
│  │  ├─ clock-in/
│  │  │  ├─ ClockInButton.tsx
│  │  ├─ clock-out/
│  │  │  ├─ ClockOutForm.tsx
│  │  ├─ TimeEntryRouter.tsx -> Decides whether to go to /clock-in or /clock-out
├─ login/
│  ├─ LoginForm.tsx

components/
├─ icons/
│  ├─ Banana.tsx
│  ├─ Logo.tsx
├─ Button.tsx
├─ Clock.tsx
├─ Container.tsx
├─ Footer.tsx
├─ Header.tsx
├─ LoadingBanana.tsx
├─ LogoutButton.tsx
├─ Nav.tsx
├─ SelectInput.tsx
├─ TextInput.tsx
```