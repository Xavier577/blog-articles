## Using absolute paths with typescript.

# Intro

Working extensively with typescript for quite some time now being someone who adopted typescript at an early stage (just 4 months after learning Javascript). I have learned and benefited a lot from the features typescript offers, one of them is being able to use absolute paths to import modules. This feature is also available in Javascript as well but in this guide, I would mostly be talking about Typescript (no harsh feeling JS) and my IDE of choice is Vscode which this feature works without any additional configuration, In other editors like WebStorm, you may have to do some extra configurations to make this work more on that in this article written by [Aman Kumar](https://medium.com/geekculture/making-life-easier-with-absolute-imports-react-in-javascript-and-typescript-bbdab8a8a3a1).

# The tsconfig.json file
If you have worked with typescript for some amount of time, you would most likely have encountered this file. Similar to the package.json file or a docker file (for docker configs), this is where the configurations for your typescript compilation are written. You can generate this file with the `tsc --init` command but most frameworks using typescript would package this file for you.

# Taking a look at relative paths 
We'll be taking a look at a nextJs project and see the horrors of using relative paths.

### Our folder structure
```
.
├── README.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── src
│   ├── components
│   │   ├── cards
│   │   │   ├── featured-motion-card.tsx
│   │   │   └── testimonal-card.tsx
│   │   ├── footer
│   │   │   └── index.tsx
│   │   ├── icon-components
│   │   │   ├── app-icon.tsx
│   │   │   ├── instagram-icon.tsx
│   │   │   ├── medium-icon.tsx
│   │   │   ├── quoteLeft.tsx
│   │   │   ├── twitter-icon.tsx
│   │   │   ├── verified-icon.tsx
│   │   │   └── youtube-icon.tsx
│   │   ├── layouts
│   │   │   └── index.tsx
│   │   └── navbar.tsx
│   ├── pages
│   │   ├── _app.tsx
│   │   ├── api
│   │   │   └── hello.ts
│   │   ├── auth
│   │   │   ├── sign-in.tsx
│   │   │   └── sign-up.tsx
│   │   ├── communities
│   │   │   └── index.tsx
│   │   ├── index.tsx
│   │   └── live-events
│   │       └── index.tsx
│   ├── shared
│   │   ├── hooks
│   │   │   ├── useActiveLinks.ts
│   │   │   └── useForm.ts
│   │   └── utils
│   ├── styles
│   │   ├── Home.module.css
│   │   └── globals.css
│   ├── theme
│   │   ├── components
│   │   ├── foundations
│   │   │   ├── colors.ts
│   │   │   └── index.ts
│   │   ├── index.ts
│   │   └── styles.ts
│   └── types
│       └── index.ts
├── tsconfig.json
└── yarn.lock
```

```
/* src/pages/communities/index.ts */
import { NextPage } from "next";
import Head from "next/head";
import { Fragment } from "react";
import {
  Box,
  Button,
  Flex,
  Heading,
  VStack,
  InputGroup,
  InputLeftElement,
  Input,
  HStack,
  Text,
  Image,
} from "@chakra-ui/react";
import { SearchIcon } from "@chakra-ui/icons";
import AppFooter from "../../components/footer"; // relative paths
import NavBar from "../../components/navbar"; // relative paths

const Communities: NextPage = () => {
   // page code
};

export default Communities;

```
This can get really messy especially when we have deeply nested files.

```
import somthing from "../../../../../somewhere/something";
```

## Configuring Typescript to use absolute paths
To enable Typescript to make use of absolute paths to import modules, first go into your tsconfig.json file.

```
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

- set the baseUrl option to `"."`

```
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
   "baseUrl": "." // here
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

- add the paths of the folders you'll want to be able to reference in the `"paths"` option, I'll be adding my `"components"`, `" theme"` and `" shared"` to the path.

```
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "baseUrl": ".", // here
    "paths": {
      "@component/*": ["./src/component/*"],
      "@theme/*": ["./src/theme/*"],
      "@shared/*": ["./src/shared/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}

```
Now we can reference our absolute path using the keys we used to reference it in our code base.

```
/* src/pages/communities/index.ts */

import { NextPage } from "next";
import Head from "next/head";
import { Fragment } from "react";
import {
  Box,
  Button,
  Flex,
  Heading,
  VStack,
  InputGroup,
  InputLeftElement,
  Input,
  HStack,
  Text,
  Image,
} from "@chakra-ui/react";
import { SearchIcon } from "@chakra-ui/icons";
import AppFooter from "@components/footer"; // absolute paths
import NavBar from "@components/navbar"; // absolute paths

const Communities: NextPage = () => {
   // page code
};

export default Communities;
```
## why use the `@` prefix
This is largely a matter of personal preference, and your reference does not have to begin with `'@'`; it can simply be the name of your file, or you can call it whatever you want as long as it is an alpha-numeric character or a string that can be a valid JSON key.

## Why use absolute paths
This is a subtle feature, but it can help with code readability and overall cleaner imports.


## Conclusion
Typescript is a wonderful programming language with tons of great features. Absolute paths have no effect on your code in runtime; it's more of a code readability thing. There are other features that do add a lot to your code, such as decorators, enums, abstract classes, and so on. This is only scratching the surface.