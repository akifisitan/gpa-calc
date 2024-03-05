# gpa-calc

An easy-to-use tool that enables Sabanci University students to calculate their semester and overall GPAs on the fly.

![Demo Image](https://github.com/akifisitan/gpa-calc/blob/main/gpa-calc.png)

## Motivation

I wanted a decent interface for calculating future course grades.

## Features

- Uses the A, A-, B+ ... D, F grading system.
- Supports multiple profiles.
- Supports Satisfactory (S), Unsatisfactory (U) grading and repeated courses.

## Built with

- [SvelteKit](https://kit.svelte.dev/)
- [Tailwind](https://tailwindcss.com/)
- [shadcn-svelte](https://www.shadcn-svelte.com)

## Developing

Install dependencies with `pnpm install`, then start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

Create a production version of the app:

```bash
npm run build
```

Preview the production build with `npm run preview`.
