# Styling

## Styling the body

- We will be using the [Pancake Stack](https://web.dev/patterns/layout/pancake-stack/)
- Edit the `tailwd.config.ts`

```ts
import type { Config } from 'tailwindcss'

const config: Config = {
    // ...
      theme: {
            extend: {
                  gridTemplateRows: {
                        pancake: "auto 1fr auto",
                  },
            },
      },
    // ...
}

export default config
```

- Now it can be used like this

```html
<div className="grid grid-rows-pancake min-h-screen">
    <section id="header"> 
        Header 
    </section>
    
    <main id="main-content"> 
        Main Content 
    </main>
    
    <section id="footer"> 
        Footer 
    </section>
</div>
```

- I really like this because there's no configuration needed on the `header`, `footer` or `main-content`, everything just works

## Styling Buttons

- Here is how I decided to code the button:

```tsx
"use client";
import React from "react";
import LeapFrogLoader from "@/components/LeapFrogLoader";
import classnames from "classnames";

export enum Type {
  BUTTON = "button",
  SUBMIT = "submit",
  RESET = "reset",
}

export enum Kind {
  SOLID = "solid",
  OUTLINE = "outline",
}

export enum Size {
  FULL = "full",
  SMALL = "small",
}

type ButtonProps = {
  children: React.ReactNode;
  onClick?: () => void;
  isLoading?: boolean;
  type?: `${Type}`;
  kind?: `${Kind}`;
  size?: `${Size}`;
};

const defaultProps = {
  isLoading: false,
  type: Type.BUTTON,
  kind: Kind.SOLID,
  size: Size.FULL,
};

const Button: React.FC<ButtonProps> = (_props) => {
  const props = { …defaultProps, …_props };

  const className = classnames(
    "block py-3 text-center rounded-lg px-4 transition ring-primary active:scale-95 shadow-0 shadow-primary",
    "hover:ring hover:ring-offset-2 hover:shadow-lg hover:shadow-primary",
    "focus:ring focus:ring-offset-2 focus:shadow-lg focus:shadow-primary",
    {
      "w-full font-semibold": props.size === Size.FULL,
    },
    {
      "bg-black text-white": props.kind === Kind.SOLID,
      "bg-white border-2 border-black text-black": props.kind === Kind.OUTLINE,
    }
  );

  return (
    <>
      <button className={className} type={props.type} onClick={props.onClick}>
        <span className="flex item-center justify-center">
          {props.isLoading && <LeapFrogLoader />}
          {props.children}
        </span>
      </button>
    </>
  );
};

export default Button;

```

### Important Notes

- button type should be `"button"` by default
    - Since having it type `"submit"` by default is usually not the intended behavior (even though the raw button element is `"submit"` by default)
- Very personal opinion: Only make the Button Component (and other components too) as generic as it needs to be for the layout you are building
    - From the Figma, we have only two types of buttons
        - Big Black Button `Kind.Solid, Size.Full`
        - Small Outlined Button for Logout: `Kind.Outline, Size.Small`
    - This means that there will be 2 more buttons that will NOT BE USED:
        - `Kind.Outline, Size.Full`
        - `Kind.Solid, Size.Small`
    - I could have instead created two separate components- 1 for the Big Black Button, and 1 for the Small Outlined One, and that would be completely fine as well
    - If we had more than one button looking `anchor` tag, I would have also put some configuration for that as well
- Deconstructing props or not is a personal choice, just be sure to follow the same convention throughout the codebase (not a strict rule, just in general)

```tsx
// deconstructed props
const Click: React.FC<ClickInterface> = ({
  children,
  className,
  size = 'md',
  type = 'button',
  color = 'primary',
  href = '',
  onClick,
}) => {
    // ...
}

// not-deconstructed props
const Click: React.FC<ClickInterface> = (props) => {
    // …
}
```