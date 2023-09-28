# Setup React Query

- [React Query v4 Official Website](https://tanstack.com/query/v4/docs/react/overview)
- [React Query NextJS 13 App Directory Docs](https://tanstack.com/query/v4/docs/react/guides/ssr#using-the-app-directory-in-nextjs-13)
- Note: I am only going to use React Query Client Side, I'm not going to deal with getting React Query to work on the server
- In a pure React App, you would setup React Query like this:

```tsx
// App.tsx

import {
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'
import { getTodos, postTodo } from '../my-api'

// Create a client
const queryClient = new QueryClient()

function App() {
  return (
    // Provide the client to your App
    <QueryClientProvider client={queryClient}>
      <TodosServerComponent />
    </QueryClientProvider>
  )
}

export default App;
```

- However, we cannot do this the same way in NextJS because the line `const queryClient = new QueryClient()` uses state and therefore cannot be put into the `RootLayout` Component since it is a server component

```tsx
// RootLayout.tsx
// DOES NOT WORK!
import {
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'
import { getTodos, postTodo } from '../my-api'

// This line uses state and thus CANNOT be used in RootLayout.tsx
const queryClient = new QueryClient()

function RootLayout() {
  return (
    // Provide the client to your App
    <QueryClientProvider client={queryClient}>
      <TodosServerComponent />
    </QueryClientProvider>
  )
}

export default RootLayout;
```

- Note that the `QueryClientProvider` is not the issue - it is a Client Component that does not import any of our Server Components, and instead just returns `<> {children} </>` where we can put our server components in - this is in line with our rules
- So to fix this, we can create another Client Component in which call this line `const queryClient = new QueryClient()` and import that component into `RootLayout.tsx` - this is exactly what the React Query docs say

```tsx
// app/providers.jsx
'use client'

import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

export default function Providers({ children }) {
  const [queryClient] = React.useState(() => new QueryClient())

  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  )
}

// app/layout.jsx
import Providers from './providers'

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <head />
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

