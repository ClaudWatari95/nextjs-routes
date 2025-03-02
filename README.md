# Next.js Routes

<blockquote>Type safe routing for Next.js</blockquote>

<br />

<a href="https://www.npmjs.com/package/nextjs-routes">
  <img src="https://img.shields.io/npm/v/nextjs-routes.svg">
</a>
<a href="https://github.com/tatethurston/nextjs-routes/blob/main/LICENSE">
  <img src="https://img.shields.io/npm/l/nextjs-routes.svg">
</a>
<a href="https://www.npmjs.com/package/nextjs-routes">
  <img src="https://img.shields.io/npm/dy/nextjs-routes.svg">
</a>
<a href="https://github.com/tatethurston/nextjs-routes/actions/workflows/ci.yml">
  <img src="https://github.com/tatethurston/nextjs-routes/actions/workflows/ci.yml/badge.svg">
</a>
<a href="https://codecov.io/gh/tatethurston/nextjs-routes">
  <img src="https://img.shields.io/codecov/c/github/tatethurston/nextjs-routes/main.svg?style=flat-square">
</a>

<br />

![nextjs-routes preview gif](./images/nextjs-routes.gif)

## What is this? 🧐

`nextjs-routes` makes Next.js's `next/link` and `next/router` routes type safe with zero runtime overhead. `nextjs-routes` scans your `pages` and/or `app` directory and generates route types based on your application's routes.

`nextjs-routes` drops into your existing Next.js application with minimal configuration. You won't have to change any code, unless it finds some broken links!

## Highlights

🦄 Zero config

💨 Types only -- zero runtime

🛠 No more broken links

🪄 Route autocompletion

🔗 Supports all Next.js route types: static, dynamic, catch all and optional catch all

## Installation & Usage 📦

1. Add this package to your project:

   `npm install nextjs-routes` or `yarn add nextjs-routes`

2. Update your `next.config.js`:

   ```diff
   + const withRoutes = require("nextjs-routes/config")();

   /** @type {import('next').NextConfig} */
   const nextConfig = {
     reactStrictMode: true,
   };

   - module.exports = nextConfig;
   + module.exports = withRoutes(nextConfig);
   ```

   File watching will only run in Next.js' development server (eg `npx next dev`).

3. Start your next server:

   `npx next dev` or `yarn next dev`

That's it! A `nextjs-routes.d.ts` file will be generated the first time you start your server. Check this file into version control. `next/link` and `next/router` type definitions have been augmented to verify your application's routes. No more broken links, and you get route autocompletion 🙌.

Whenever your routes change, your `nextjs-routes.d.ts` file will automatically update.

## Examples 🛠

### Link

`Link`'s `href` prop is now typed based on your application routes:

```tsx
import Link from "next/link";

<Link
  href={{
    pathname: "/foos/[foo]",
    query: { foo: "bar" },
  }}
>
  Bar
</Link>;
```

If the route doesn't require any parameters, you can also use a path string:

```tsx
<Link href="/foo">Foo</Link>
```

### useRouter

`useRouter`'s returned router instance types for `push`, `replace` and `query` are now typed based on your application routes.

Identical to `Link`, `push` and `replace` now expect a UrlObject or path string:

#### push

```tsx
import { useRouter } from "next/router";

const router = useRouter();
router.push({ pathname: "/foos/[foo]", query: { foo: "test" } });
```

#### replace

```tsx
import { useRouter } from "next/router";

const router = useRouter();
router.replace({ pathname: "/" });
```

#### query

```tsx
import { useRouter } from "next/router";

// query is typed as a union of all query parameters defined by your application's routes
const { query } = useRouter();
```

By default, `query` will be typed as the union of all possible query parameters defined by your application routes. If you'd like to narrow the type to fewer routes or a single page, you can supply a type argument:

```tsx
import { useRouter } from "next/router";

// query is now typed as `{ foo: string }`
const { query } = useRouter<"/foos/[foo]">();
```

### Route

If you want to use the generated `Route` type in your code, you can import it from `nextjs-routes`:

```ts
import type { Route } from "nextjs-routes";
```

### RoutedQuery

If you want to use the generated `Query` for a given `Route`, you can import it from `nextjs-routes`:

```ts
import type { RoutedQuery } from "nextjs-routes";
```

This is useful as the context type parameter inside `getStaticProps` and `getServerSideProps`:

```ts
import type { GetStaticProps } from "next";
import type { RoutedQuery } from "nextjs-routes";

export const getStaticProps: GetStaticProps<{}, RoutedQuery<"/foos/[foo]">> = (
  context
) => {
  // context.params will include `foo` as a string
  const { foo } = context.params;
};
```

## How does this work? 🤔

`nextjs-routes` generates types for the `pathname` and `query` for every page in your `pages` and/or `app` directory. The generated types are written to `nextjs-routes.d.ts` which is automatically referenced by your Next project's `tsconfig.json`. `nextjs-routes.d.ts` redefines the types for `next/link` and `next/router` and applies the generated route types.

## What if I need a runtime?

There are some cases where you may want to generate a type safe path from a `Route` object, such as when `fetch`ing from an API route or serving redirects from `getServerSideProps`. These accept `strings` instead of the `Route` object that `Link` and `useRouter` accept. Because these do not perform the same string interpolation for dynamic routes, runtime code is required instead of a type only solution.

For these cases, you can use `route` from `nextjs-routes`:

### fetch

```ts
import { route } from "nextjs-routes";

fetch(route({ pathname: "/api/foos/[foo]", query: { foo: "foobar" } }));
```

### getServerSideProps

```ts
import { route } from "nextjs-routes";

export const getServerSideProps: GetServerSideProps = async (context) => {
  return {
    redirect: {
      destination: route({ pathname: "/foos/[foo]", query: { foo: "foobar" } }),
      permanent: false,
    },
  };
};
```

### Internationalization (i18n)

`nextjs-routes` refines `Link` and `useRouter` based on your [Nextjs i18n configuration](https://nextjs.org/docs/advanced-features/i18n-routing).

The following `next.config.js`:

```js
module.exports = withRoutes({
  i18n: {
    defaultLocale: "de-DE",
    locales: ["de-DE", "en-FR", "en-US"],
  },
});
```

Will type `Link` and `useRouter`'s `locale` as `'de-DE' | 'en-FR' | 'en-US'`. All other i18n properties (`defaultLocale`, `domainLocales` and `locales`) are also typed.

If you want to use the generated `Locale` type, you can import it from `nextjs-routes`:

```ts
import { Locale } from "nextjs-routes";
```

## Configuration

You can pass the following options to `withRoutes` in your `next.config.js`:

```js
const withRoutes = require("nextjs-routes/config")({
  outDir: "types",
});
```

- `outDir`: The file path indicating the output directory where the generated route types should be written to (e.g.: "types"). The default is to create the file in the same folder as your `next.config.js` file.

## Contributing 👫

PR's and issues welcomed! For more guidance check out [CONTRIBUTING.md](https://github.com/tatethurston/nextjs-routes/blob/main/CONTRIBUTING.md)

Are you interested in bringing a `nextjs-routes` like experience to another framework? [Open an issue](https://github.com/tatethurston/nextjs-routes/issues/new) and let's collaborate.

## Licensing 📃

See the project's [MIT License](https://github.com/tatethurston/nextjs-routes/blob/main/LICENSE).
