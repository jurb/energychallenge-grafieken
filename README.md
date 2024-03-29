Graphs for the Energy Challenge prototype. Takes in data via URL parameters.

Demo:

- <https://challenge-grafieken-2019.focustest.nl/?data=23&kind=stroom>
- <https://challenge-grafieken-2019.focustest.nl/?data=23,34,21,17&kind=stroom>
- <https://challenge-grafieken-2019.focustest.nl/?data=23,34,35,21,17&kind=stroom>
- <https://challenge-grafieken-2019.focustest.nl/?data=23,34,35,21,17&kind=gas>
- <https://challenge-grafieken-2019.focustest.nl/?data=23,34,35,21,17&kind=stroom>
- <https://challenge-grafieken-2019.focustest.nl/?data=23,34,35,21,17&kind=gas&charttype=endchart1v3>
- <https://challenge-grafieken-2019.focustest.nl/?data=0,0,0,0,0,0,.1,.4,.6,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0&kind=gas&charttype=daychart&date=4%20september>

# svelte app

This is a project template for [Svelte](https://svelte.dev) apps. It lives at https://github.com/sveltejs/template.

To create a new project based on this template using [degit](https://github.com/Rich-Harris/degit):

```bash
npx degit sveltejs/template svelte-app
cd svelte-app
```

_Note that you will need to have [Node.js](https://nodejs.org) installed._

## Get started

Install the dependencies...

```bash
cd svelte-app
npm install
```

...then start [Rollup](https://rollupjs.org):

```bash
npm run dev
```

Navigate to [localhost:5000](http://localhost:5000). You should see your app running. Edit a component file in `src`, save it, and reload the page to see your changes.

## Deploying to the web

### With [now](https://zeit.co/now)

Install `now` if you haven't already:

```bash
npm install -g now
```

Then, from within your project folder:

```bash
cd public
now
```

As an alternative, use the [Now desktop client](https://zeit.co/download) and simply drag the unzipped project folder to the taskbar icon.

### With [surge](https://surge.sh/)

Install `surge` if you haven't already:

```bash
npm install -g surge
```

Then, from within your project folder:

```bash
npm run build
surge public
```
