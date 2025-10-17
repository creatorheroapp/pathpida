# pathpida

<br />
<img src="https://aspida.github.io/pathpida/logos/png/logo.png" alt="pathpida" title="pathpida" />
<div align="center">
  <a href="https://www.npmjs.com/package/pathpida">
    <img src="https://img.shields.io/npm/v/pathpida" alt="npm version" />
  </a>
  <a href="https://www.npmjs.com/package/pathpida">
    <img src="https://img.shields.io/npm/dm/pathpida" alt="npm download" />
  </a>
</div>
<br />
<p align="center">TypeScript friendly pages and static path generator for Next.js.</p>
<br />
<br />

## Breaking change :warning:

### 2024/12/14

Since pathpida >= `0.23.0` , removed Nuxt support.

### 2022/11/25

Since pathpida >= `0.20.0` , removed Sapper support.

## Features

- **Type safety**. Automatically generate type definition files for manipulating internal links in Next.js.
- **Zero configuration**. No configuration required can be used immediately after installation.
- **Zero runtime**. Lightweight because runtime code is not included in the bundle.
- **Support for static files**. Static files in public/ are also supported, so static assets can be safely referenced.
- **Support for appDir of Next.js 13 Layout**.

## Table of Contents

- [Install](#Install)
- [Command Line Interface Options](#CLI-options)
- [Setup](#Setup)
- [Usage](#Usage)
- [Define query](#Define-query)
- [Generate static files path](#Generate-static-files-path)
- [License](#License)

## Install

- Using [npm](https://www.npmjs.com/):

  ```sh
  $ npm install pathpida npm-run-all --save-dev
  ```

- Using [Yarn](https://yarnpkg.com/):

  ```sh
  $ yarn add pathpida npm-run-all --dev
  ```

<a id="CLI-options"></a>

## Command Line Interface Options

<table>
  <thead>
    <tr>
      <th>Option</th>
      <th>Type</th>
      <th width="100%">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td nowrap><code>--enableStatic</code><br /><code>-s</code></td>
      <td></td>
      <td>Generate static files path in <code>$path.ts</code>.</td>
    </tr>
    <tr>
      <td nowrap><code>--ignorePath</code><br /><code>-p</code></td>
      <td><code>string</code></td>
      <td>Specify the ignore pattern file path.</td>
    </tr>
    <tr>
      <td nowrap><code>--ignoreAppSegments</code><br /><code>-i</code></td>
      <td><code>string</code></td>
      <td>Comma-separated list of folder names to skip in URL generation at any level of the app directory. The structure inside these folders will still be read and processed (e.g., <code>[locale]</code> for i18n routing handled by middleware).</td>
    </tr>
    <tr>
      <td nowrap><code>--output</code><br /><code>-o</code></td>
      <td><code>string</code></td>
      <td>Specify the output directory for <code>$path.ts</code>.</td>
    </tr>
    <tr>
      <td nowrap><code>--watch</code><br /><code>-w</code></td>
      <td></td>
      <td>
        Enable watch mode.<br />
        Regenerate <code>$path.ts</code>.
      </td>
    </tr>
    <tr>
      <td nowrap><code>--version</code><br /><code>-v</code></td>
      <td></td>
      <td>Print pathpida version.</td>
    </tr>
  </tbody>
</table>

## Setup

`package.json`

```json
{
  "scripts": {
    "dev": "run-p dev:*",
    "dev:next": "next dev",
    "dev:path": "pathpida --ignorePath .gitignore --watch",
    "build": "pathpida --ignorePath .gitignore && next build"
  }
}
```

### Ignoring App Router Segments

If you're using Next.js 13+ app router with internationalization or other patterns where you want to skip specific folders in URL generation (like `[locale]`), you can use the `--ignoreAppSegments` flag. This will **read the structure inside these folders** but **skip them when building URLs**.

This works at **any level** of your app directory structure, not just at the root.

```json
{
  "scripts": {
    "dev": "run-p dev:*",
    "dev:next": "next dev",
    "dev:path": "pathpida --ignoreAppSegments [locale] --watch",
    "build": "pathpida --ignoreAppSegments [locale] && next build"
  }
}
```

You can ignore multiple segments by separating them with commas:

```json
{
  "scripts": {
    "dev:path": "pathpida --ignoreAppSegments [locale],[region] --watch"
  }
}
```

#### Example 1: Root-level locale

```
src/app/[locale]/dashboard/page.tsx
src/app/[locale]/profile/page.tsx
```

The pages inside `[locale]` will be read, but the `[locale]` segment will be skipped:

```typescript
pagesPath.dashboard.$url(); // Generates { pathname: '/dashboard' }
pagesPath.profile.$url(); // Generates { pathname: '/profile' }

// Instead of:
// pagesPath._locale('en').dashboard.$url() // Would generate { pathname: '/[locale]/dashboard', query: { locale: 'en' } }
```

#### Example 2: Nested locale (works at any level)

```
src/app/admin/[locale]/dashboard/page.tsx
src/app/public/[locale]/help/page.tsx
```

With `--ignoreAppSegments [locale]`:

```typescript
pagesPath.admin.dashboard.$url(); // Generates { pathname: '/admin/dashboard' }
pagesPath.public.help.$url(); // Generates { pathname: '/public/help' }
```

This is particularly useful when:

- Your locale is handled by Next.js middleware and shouldn't be in the URL
- You want to avoid extra redirects that impact performance
- The dynamic segment is a routing implementation detail that shouldn't appear in your path helpers

## Usage

```
pages/index.tsx
pages/post/create.tsx
pages/post/[pid].tsx
pages/post/[...slug].tsx

lib/$path.ts or utils/$path.ts // Generated automatically by pathpida
```

or

```
src/pages/index.tsx
src/pages/post/create.tsx
src/pages/post/[pid].tsx
src/pages/post/[...slug].tsx

src/lib/$path.ts or src/utils/$path.ts // Generated automatically by pathpida
```

`pages/index.tsx`

```tsx
import Link from 'next/link';
import { pagesPath } from '../lib/$path';

console.log(pagesPath.post.create.$url()); // { pathname: '/post/create' }
console.log(pagesPath.post._pid(1).$url()); // { pathname: '/post/[pid]', query: { pid: 1 }}
console.log(pagesPath.post._slug(['a', 'b', 'c']).$url()); // { pathname: '/post//[...slug]', query: { slug: ['a', 'b', 'c'] }}

export default () => {
  const onClick = useCallback(() => {
    router.push(pagesPath.post._pid(1).$url());
  }, []);

  return (
    <>
      <Link href={pagesPath.post._slug(['a', 'b', 'c']).$url()} />
      <div onClick={onClick} />
    </>
  );
};
```

<a id="Define-query"></a>

## Define query

`pages/post/create.tsx`

```tsx
export type Query = {
  userId: number;
  name?: string;
};

export default () => <div />;
```

`pages/post/[pid].tsx`

```tsx
export type OptionalQuery = {
  limit: number;
  label?: string;
};

export default () => <div />;
```

`pages/index.tsx`

```tsx
import Link from 'next/link';
import { pagesPath } from '../lib/$path';

console.log(pagesPath.post.create.$url({ query: { userId: 1 } })); // { pathname: '/post/create', query: { userId: 1 }}
console.log(pagesPath.post.create.$url()); // type error
console.log(pagesPath.post._pid(1).$url()); // { pathname: '/post/[pid]', query: { pid: 1 }}
console.log(pagesPath.post._pid(1).$url({ query: { limit: 10 }, hash: 'sample' })); // { pathname: '/post/[pid]', query: { pid: 1, limit: 10 }, hash: 'sample' }

export default () => {
  const onClick = useCallback(() => {
    router.push(pagesPath.post._pid(1).$url());
  }, []);

  return (
    <>
      <Link href={pagesPath.post._slug(['a', 'b', 'c']).$url()} />
      <div onClick={onClick} />
    </>
  );
};
```

<a id="Generate-static-files-path"></a>

## Generate static files path

`package.json`

```json
{
  "scripts": {
    "dev": "run-p dev:*",
    "dev:next": "next dev",
    "dev:path": "pathpida --enableStatic --watch",
    "build": "pathpida --enableStatic && next build"
  }
}
```

```
pages/index.tsx
pages/post/create.tsx
pages/post/[pid].tsx
pages/post/[...slug].tsx

public/aa.json
public/bb/cc.png

lib/$path.ts or utils/$path.ts // Generated automatically by pathpida
```

or

```
src/pages/index.tsx
src/pages/post/create.tsx
src/pages/post/[pid].tsx
src/pages/post/[...slug].tsx

public/aa.json
public/bb/cc.png

src/lib/$path.ts or src/utils/$path.ts // Generated automatically by pathpida
```

`pages/index.tsx`

```tsx
import Link from 'next/link';
import { pagesPath, staticPath } from '../lib/$path';

console.log(staticPath.aa_json); // /aa.json

export default () => {
  return (
    <>
      <Link href={pagesPath.post._slug(['a', 'b', 'c']).$url()} />
      <img src={staticPath.bb.cc_png} />
    </>
  );
};
```

## License

pathpida is licensed under a [MIT License](https://github.com/aspida/pathpida/blob/main/LICENSE).
