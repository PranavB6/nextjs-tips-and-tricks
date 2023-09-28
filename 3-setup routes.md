# Setup Routes

- We know that our app with have a `login` page, a `time-entry` page, and a `history` page

```
app/
├─ /    -> login
├─ history/
├─ time-entry/

```

- If a user is logged in, we want to take them to the `login` page, otherwise, we want to take them to the `time-entry` page. We could handing this logic in the login page, but I think it would be a better separation of concerns if we handle this in the `/` "home" page

```
app/
├─ /
│  ├─ page.tsx    -> if a user is logged in, redirect to /time-entry, otherwise /login
├─ history/
├─ login/
├─ time-entry/

```

- When the user clocks in vs when they clock out, there is very different logic and also looks different. Let's create two different routes under time-entry for that.

```
app/
├─ /
│  ├─ page.tsx    -> if a user is logged in, redirect to /time-entry, otherwise /login
├─ history/
├─ login/
├─ time-entry/
│  ├─ clock-in/
│  ├─ clock-out/
```

- Another advantage of of having separate pages for `clock-in` and `clock-out` is that we put the logic of determining whether a user should see the `clock-in` page or the `clock-out` page in the `page.tsx` file of the `time-entry` page

```
app/
├─ /
│  ├─ page.tsx    -> if a user is logged in, redirect to /time-entry, otherwise /login
├─ history/
├─ login/
├─ time-entry/
│  ├─ clock-in/
│  ├─ clock-out/
│  ├─ page.tsx    -> if user is clocked-in, redirect to /clock-out, otherwise /clock-in

```

- Since we know that the `time-entry` page and the `history` page will share the same layout, it would be good if we could somehow "group" them together. Luckily, NextJS has a feature just for this called [Route Groups](https://nextjs.org/docs/app/building-your-application/routing/route-groups). If you created a folder with parentheses, then it prevents the folder from being included in the route's URL path, but the `layout.tsx` will still be shared with the routes / folders under it

```
app/
├─ /
│  ├─ page.tsx    -> if a user is logged in, redirect to /time-entry, otherwise /login
├─ (protected)/
│  ├─ history/
│  ├─ time-entry/
│  │  ├─ clock-in/
│  │  ├─ clock-out/
│  │  ├─ page.tsx  -> if user is clocked-in, redirect to /clock-out, otherwise /clock-in
│  ├─ layout.tsx   -> shared layout between /history and /time-entry
├─ login/


```

- Filling in the rest of the `page.tsx`s and `layout.tsx`, the final folder structure will look like this:

```
app/
├─ (protected)/
│  ├─ history/
│  │  ├─ page.tsx
│  ├─ time-entry/
│  │  ├─ clock-in/
│  │  │  ├─ page.tsx
│  │  ├─ clock-out/
│  │  │  ├─ page.tsx
│  │  ├─ page.tsx
│  ├─ layout.tsx
├─ login/
│  ├─ layout.tsx
│  ├─ page.tsx
├─ layout.tsx
├─ page.tsx

```