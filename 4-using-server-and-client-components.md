# Using Server and Client Component Together

- [Reference](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns)

## Server / Client Composition Patterns

### Using a Client Component inside a Server Component

- This works the same as usual, you can use and import components as normal

```jsx
// server-component
import ClientComponent from "./client-component"

export default function ServerComponent() {
    return (
        <>
            <ClientComponent />
        </>
    )
}
```

### Using a Server Component inside a Client Component

- You **cannot** do this the "normal" way, only if the Client Component accepts a children prop can you put a Server Component inside

```jsx
// client-component
"use-client"

// YOU CANNOT IMPORT A SERVER COMPONENT INSIDE A CLIENT COMPONENT

export default ClientComponent({ children }) {

    return (
        <>
            { children }
        </>
    )

}
```

```jsx
// page
import ClientComponent from "./client-component"
import ServerComponent from "./server-component"

export default function Page() {
    return (
        <>
            <ClientComponent>
                <ServerComponent />
            </ClientComponent>
        </>
    )
}
```

## Enforcing Server Components

- Install the `server-only` package

```bash
pnpm add server-only
```

- Import the package into any module that contains server-only code

```js
import "server-only"

export async function ServerComponent {
    // ...
}
```

- This will throw an error if you tried to execute server-only code on the client

### Using 3rd Party Components

1. Most 3rd party npm libraries do not yet have the `"use client"` directive
2. Any code executed in a client component will only be executed on the client, including imported code

- Together with these two rules, you just have to make sure that you only use 3rd party libraries in your own client components, and there will be no issues

## Special Cases - Clock Component

- Madeeha, you have already discovered this, but I wanted to write it down to further cement the understanding
- A simple Clock Component may look like this:

```tsx

"use client";
import React, { useEffect, useState } from "react";

const getCurrentTime = () => {
  return new Date().toLocaleTimeString();
};

const Clock: React.FC = () => {
  const [currTime, setCurrTime] = useState(getCurrentTime());

  useEffect(() => {
    setCurrTime(getCurrentTime());

    const interval = setInterval(() => {
      setCurrTime(getCurrentTime());
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return <>{currTime}</>;
};

export default Clock;

```

- However, you may get an error like this:

```
Text content did not match. Server: "9:20:44 p.m." Client: "9:20:45 p.m."
```

- This happens because when the server renders the component, it has a different time than when the client renders the component
- This fix for this is very simple, make the initial state on both the client and the server the same, and only change the state on the client

```tsx
"use client";
import React, { useEffect, useState } from "react";

const getCurrentTime = () => {
  return new Date().toLocaleTimeString();
};

const Clock: React.FC = () => {
  // initial state on both client and server is ""
  const [currTime, setCurrTime] = useState("");

  useEffect(() => {
    // immediately set the current time so that the user does not have to wait a second to see the time
    setCurrTime(getCurrentTime());
  
    const interval = setInterval(() => {
      setCurrTime(getCurrentTime());
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return <>{currTime}</>;
};

export default Clock;
```