# Midpoint Finder 🔍
A way to calculate the midpoint between two people on a map, and find places to meet at that point, such as restaraunts.

## api key
To use this project, you need a google maps API key. Just create a `.env` file, and make a new variable called `PUBLIC_GOOGLE_MAPS_API_KEY`. set this variable equal to the maps api key that you have. You can make an api key for maps in the google cloud console.

## A demonstration of the app (gif is below):
![demonstration gif](https://github.com/bhargavg1/midpointfinder/blob/master/midpoint%20gif.gif)

## sv

Everything you need to build a Svelte project, powered by [`sv`](https://github.com/sveltejs/cli).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npx sv create

# create a new project in my-app
npx sv create my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.
