---
layout: post
title: "Google Tag Manager in Jekyll: One Snippet, Two Hooks, and a Build That Tried to Fight Back"
date: 2026-05-08 10:00:00 +0200
categories: [Analytics, Jekyll]
tags: [Google Tag Manager, Jekyll, Liquid, Debugging, HTML]
summary: "How I implemented Google Tag Manager in my Jekyll site, why the default layout was the right place for it, and how a broken build briefly made the tag look guilty."
---

## Why I Wanted to Add GTM

At some point, a portfolio site stops being just a collection of pages and starts becoming something I actually want to measure.

Not in a creepy "track every pixel of human emotion" way.
Just in a practical way:

- are pages loading correctly
- is traffic reaching the site
- are analytics tools actually firing

So the next step was to add **Google Tag Manager** to the Jekyll site.

## The Core Idea

Google gives two pieces of code for a standard GTM installation:

1. a script that should be placed as high as possible inside `<head>`
2. a `noscript` iframe that should be placed immediately after the opening `<body>` tag

The beginner instinct might be to paste that into every page manually.

That would work.
It would also be a wonderful way to create future chaos.

Because this is a **Jekyll** site, the cleaner solution was to place both snippets in the base layout:

`_layouts/default.html`

That way every page using the default layout inherits the GTM container automatically.

## The Implementation

The `<head>` section now includes this snippet:

```html
<!-- Google Tag Manager -->
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-M63NRDFG');</script>
<!-- End Google Tag Manager -->
```

And directly after the opening `<body>` tag, I added:

```html
<!-- Google Tag Manager (noscript) -->
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-M63NRDFG"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<!-- End Google Tag Manager (noscript) -->
```

That is the whole GTM installation from a markup perspective.

Simple.
Clean.
The kind of task that looks like it should take five minutes and never become interesting.

## Why the Default Layout Was the Right Place

This site uses shared Jekyll layouts for pages and posts, so adding GTM centrally has a few big advantages:

- one source of truth instead of editing every page manually
- less risk of forgetting the tag on future posts
- easier maintenance if the container ID ever changes
- much cleaner structure overall

This is one of those moments where using a static site generator actually feels very elegant.

Change one layout.
Update the whole site.
In theory.

## The Part Where It Got Annoying

At first, this looked like a simple "GTM is not working" problem.

But as usual, the real bug was standing somewhere else in the room pretending to be innocent.

The deeper issue turned out to be the **Jekyll build path**, not the GTM snippet itself.

While checking the implementation, I found that:

- parts of the generated `_site` preview had inconsistent markup after manual syncing
- some layouts had broken or overly fragile Liquid structure
- the local Jekyll environment was also unhappy, which made rebuilds more dramatic than they needed to be

So the lesson here was useful:

sometimes the tag manager is fine, but the site pipeline around it is not.

In other words, GTM was being blamed for a crime committed by the build process.

## What I Verified

After the implementation, the important checks were:

- the GTM script exists in the page `<head>`
- the `noscript` iframe exists right after `<body>`
- the base layout is the place that controls the behavior site-wide
- source files, not just local `_site`, contain the final implementation

That last point matters a lot on GitHub Pages, because the live site is built from source files, not from the local `_site` folder.

So if the source layout is wrong, the live result is wrong.
And if the source layout is right but the build is broken, that is a different problem entirely.

## Outcome

At the end of the implementation:

- GTM was added in the correct two locations
- the setup was centralized in the default Jekyll layout
- the site became easier to maintain going forward
- the debugging process also exposed layout and build issues that needed separate attention

## Final Thought

Adding Google Tag Manager was supposed to be a small analytics task.

It still was.

It just also became one of those classic admin moments where a "tiny change" politely removes a floorboard and shows you three other problems living underneath it.

Which, to be fair, is also a form of observability.
