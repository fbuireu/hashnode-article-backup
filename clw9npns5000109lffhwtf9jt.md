---
title: "Crafting a custom Breadcrumb component in Astro:  Step-by-Step Guide"
seoTitle: "Crafting a custom Breadcrumb component in Astro:  Step-by-Step Guide"
seoDescription: "Building your own Breadcrumbs component in Astro and TypeScript: for DIY enthusiasts"
datePublished: Thu May 16 2024 19:41:44 GMT+0000 (Coordinated Universal Time)
cuid: clw9npns5000109lffhwtf9jt
slug: crafting-a-custom-breadcrumb-component-in-astro-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/Cos6favumqA/upload/94fea077a2403ccd1a3fc4a53f0b2f56.jpeg
tags: development, seo, learning, typescript, frontend-development, astro

---

# The motivation

If you're like me, you love the thrill of building components from scratch instead of relying on third-party libraries that sometimes feel like black boxes. If that hits home, then this article is your jam. Recently, I embarked on a mission to create a Breadcrumbs component that follows JSON-LD standards, only to find a shocking lack of customizable solutions. I stumbled upon a bunch of low-maintenance libraries that didn't offer much flexibility.

I've been there, done that, so I'm pumped to share the final snippet with you right away. Go ahead, copy/paste with glee, and then join me as I break down each step.

# TLDR; The Snippet

```typescript
---
import { generateBreadcrumbs } from './utils/generateBreadcrumbs';

const { pathname:currentPath, origin }= Astro.url;

const breadcrumbs = generateBreadcrumbs({ currentPath });
---

<div>
  {breadcrumbs?.map(({ link, label }) => {
      const isLast = breadcrumbs.at(-1).link === link

      return (
        <>
          {!isLast ? (
          <a href={link}>{label}</a>
            ) : (
            <span>{label}</span>
            )}
          {!isLast && " / "}
        </>
      );
    })}
</div>

<script type="application/ld+json" set:html={JSON.stringify({
  "@context": "https://schema.org/",
  "@type": "BreadcrumbList",
  "itemListElement": breadcrumbs.map(({ link, label }, index) => {
    const isLast = breadcrumbs.at(-1).link === link
    
    return {
      "@type": "ListItem",
      "position": index + 1,
      "name": label,
      "item": isLast ? undefined : `${origin}${link}`
    };
  })
})}/>
```

Where `generateBreadcrumbs` is:

```ts
import { deSlugify } from '@shared/utils/deSlugify';

interface GenerateBreadcrumbsProps{
  currentPath: string,
}

export const generateBreadcrumbs = ({ currentPath }: GenerateBreadcrumbsProps) => {
  const pathSegments = currentPath.split('/').filter(segment => segment.trim() !== '');
  const breadcrumbs = pathSegments.map((_, index) => {
    const link = `/${pathSegments.slice(0, index + 1).join('/')}`;
    const label = deSlugify(pathSegments.at(index) ?? '');

    return { label, link };
  });

  currentPath !== '/' && breadcrumbs.unshift({ label: "Home", link: "/" });

  return breadcrumbs;
}
```

And where `deSlugify` serves as an auxiliary (and optional) function, converting a slug-like string into a standard one by:

```ts
export function deSlugify(slug: string): string {
	return slug
		.replace(/-/g, ' ')
		.replace(/\b\w/g, match => match.toUpperCase());
}
```

So, let's break it down.

# The breakdown

## `Breadcrumbs.astro` (Global Variables)

Astro makes it a breeze to access global variables in `.astro` files, including the current URL path. We just destructure the `Astro.url` object to get `pathname` (aliased as `currentPath`) and `origin` From [Astro docs](https://docs.astro.build/en/reference/api-reference/#astro-global):

> The `Astro` global is available in all contexts in `.astro` files.

```typescript
const { pathname:currentPath, origin }= Astro.url;
```

Given a URL like: `https://domain.com/blog/some-post-title`

The `currentPath` will hold the path part of the URL, while `origin` represents the domain (including the protocol, e.g.: https://domain.com).

## `generateBreadcrumbs.ts`

The `generateBreadcrumbs` function splits the `currentPath` into segments and constructs breadcrumbs from them.

```ts
interface GenerateBreadcrumbsProps{
  currentPath: string,
}

export const generateBreadcrumbs = ({ currentPath }: GenerateBreadcrumbsProps) => {
  const pathSegments = currentPath.split('/').filter(segment => segment.trim() !== '');
  const breadcrumbs = pathSegments.map((_, index) => {
    const link = `/${pathSegments.slice(0, index + 1).join('/')}`;
    const label = deSlugify(pathSegments.at(index) ?? '');

    return { label, link };
  });

  currentPath !== '/' && breadcrumbs.unshift({ label: "Home", link: "/" });

  return breadcrumbs;
}
```

<s>First line first</s> First things first: since some domain providers or configurations can return a trailing slash in the URL (like [`domain.com/blog/some-post-title/`](http://domain.com/blog/some-post-title/)), we need to clean up the URL by:

```ts
const pathSegments = currentPath.split('/').filter(segment => segment.trim() !== '');
```

Otherwise, we may be splitting a URL and getting `['', 'blog', 'some-post-title', '']` which isn't ideal.

Once this safeguard is in place, we loop through those segments and return a `label` and a `link` (I chose to return an object with `label` and `link` properties for convenience, but you can tweak it as needed).

```ts
    const link = `/${pathSegments.slice(0, index + 1).join('/')}`;
    const label = deSlugify(pathSegments.at(index) ?? '');
```

In short, `link` is basically going to each position of the `pathSegments` array `(0, index + 1)` and creating a link like: `/blog`, `/blog/some-post-title`, for each iteration.

In long:

Given the following `pathSegment` (which will be extracted from a URL like `domain.com/blog/some-post-title`).

Given: `pathSegments = ['blog', 'some-post-title']`

**First iteration (**`index` is `0`**):**

* `pathSegments.slice(0, 0 + 1)` returns `['blog']`.
    
* `.join('/')` joins the segments with the `/` character, resulting in `'/blog'`.
    

**Second iteration (**`index` is `1`**):**

* `pathSegments.slice(0, 1 + 1)` returns `['blog', 'some-post-title']`.
    
* `.join('/')` joins the segments with the `/` character, resulting in `'blog/some-post-title'`.
    

On the other hand, `label` is just returning the link in each position and passing it through the `deSlugify` auxiliar function.

Finally, if we are not in the homepage we are adding the link and the label for it via [`unshift` method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift):

```ts
currentPath !== '/' && breadcrumbs.unshift({ label: "Home", link: "/" });
```

This adds an initial position in the array for returning to the homepage.

## `deSlugify.ts`

This is a dummy function to build a readable text based on a slug-like string. Given a `some-post-title` it will return `Some Post Title` :

```typescript
export function deSlugify(slug: string): string {
  return slug
   .replace(/-/g, ' ')
   .replace(/\b\w/g, match => match.toUpperCase());
}
```

The code is self-explanatory. First, we replace all hyphens (`-`) with spaces (`' '`) and then capitalize each first letter of each word using the regex (`/\b\w/g`).

## `Breadcrumbs.astro` (The loop)

Once all the heavy-lifting is done, we just need to gather our fancy and fresh breadcrumbs by:

```typescript
const breadcrumbs = generateBreadcrumbs({ currentPath });
```

And loop through them:

```typescript
{breadcrumbs?.map(({ link, label }) => {
    const isLast = breadcrumbs.at(-1).link === link

    return (
      <>
        {!isLast ? (
        <a href={link}>{label}</a>
          ) : (
          <span>{label}</span>
          )}
        {!isLast && " / "}
      </>
    );
  })}
```

A few things to consider: we don't want to render an anchor (`<a>`) if the element isn't clickable, and the last element shouldn't be clickable if it points to the current page. Basically, if the link of the element in the last position ([`breadcrumbs.at`](http://breadcrumbs.at)`(-1)`) is the same as the current one, we know we're in the last iteration:

```typescript
const isLast = breadcrumbs.at(-1).link === link
```

[*More information about .at()*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at)

Hence, if we are in the last iteration, we render a `<span>` .

The same approach applies for rendering a separator (`/` in this case):

```typescript
  {!isLast && " / "}
```

## `Breadcrumbs.astro` (the JSON-LD)

Lastly, we construct the JSON-LD script for structured data using the breadcrumbs information.

```typescript
<script type="application/ld+json" set:html={JSON.stringify({
  "@context": "https://schema.org/",
  "@type": "BreadcrumbList",
  "itemListElement": breadcrumbs.map(({ link, label }, index) => {
    const isLast = breadcrumbs.at(-1).link === link
    
    return {
      "@type": "ListItem",
      "position": index + 1,
      "name": label,
      "item": isLast ? undefined : `${origin}${link}`
    };
  })
})}/>
```

We're taking advantage of the [`set:html` directive](https://docs.astro.build/en/reference/directives-reference/#sethtml) from Astro which allows us to inline scripts. `item` property is not required for the last item so we are ignoring it. The output following the previous example will be:

```xml
<script type="application/ld+json">
  {
    "@context": "https://schema.org/",
    "@type": "BreadcrumbList",
    "itemListElement": [
      {
        "@type": "ListItem",
        "position": 1,
        "name": "Home",
        "item": "https://domain.com"
      },
      {
        "@type": "ListItem",
        "position": 2,
        "name": "Blog",
        "item": "https://domain.com/blog"
      },
      {
        "@type": "ListItem",
        "position": 3,
        "name": "Some Post Title"
      }
    ]
  }
</script>
```

## The use

```typescript
<Breadcrumbs />
```

No kidding. Simple as that. Easy as pie

![Video gif. A lizard's body is moving up and down in conjunction to its mouth as it laughs. Text, "hehehehehe."](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExbXV2eHdnYnkwaDV4dzI0bXZoYXg1dDhpaGxiYnB3bW9vMTg0aXc3aSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/9MFsKQ8A6HCN2/giphy.gif align="left")

# Conclusion

Creating a custom Breadcrumb component in Astro not only improves your website's navigation but also ensures it meets JSON-LD standards, boosting SEO. By walking through each step, from handling global variables to constructing the JSON-LD script, we've built a fully customizable and efficient solution.

I hope this guide inspires you to create more bespoke components, giving you the flexibility and control that third-party libraries often lack.

Happy coding!