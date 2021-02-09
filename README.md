*Looking for a shareable component template? Go here --> [sveltejs/component-template](https://github.com/sveltejs/component-template)*

---

# Svelte app
A game "guess the price" built with svelte

## Initial setup
```bash
npx degit sveltejs/template svelte-app
cd svelte-app
cd ~/path/to/project # go to project dir
npm install # install dependencies
```

## Develop
- The bundler which is used - [Rollup](https://rollupjs.org):
```bash
npm run dev # run the dev
```
- By default, the server will only respond to requests from localhost. To allow connections from other computers,
  edit the `sirv` commands in package.json to include the option `--host 0.0.0.0`


## Build app
```bash
npm run build # build for production
```
- You can run the newly built app with `npm run start`. This uses
[sirv](https://github.com/lukeed/sirv), which is included in your `package.json` dependencies so
that the app will work when you deploy to platforms like [Heroku](https://heroku.com).


## Single-page app mode
- By default, sirv will only respond to requests that match files in `public`. This is to maximise
  compatibility with static fileservers, allowing you to deploy your app anywhere.
- If you're building a single-page app (SPA) with multiple routes, sirv needs to be able to respond
  to requests for *any* path. You can make it so by editing the `"start"` command in package.json:
```js
"start": "sirv public --single"
```


## Deploying to the web
### With [now](https://zeit.co/now)
```bash
npm install -g now # install "now"
cd ~/path/to/project # go to project dir
now deploy --name my-project # deploy to now
```
- As an alternative, use the [Now desktop client](https://zeit.co/download) and simply drag the 
  unzipped project folder to the taskbar icon.

### With [surge](https://surge.sh/)
```bash
npm install -g surge # install surge
npm run build # build app for production
surge public my-project.surge.sh # deploy to surge
```

## Notes
- Reactive declarations needed for the derived state
```sveltehtml
<script>
	let count = 0;
	$: doubled = count * 2;

	function handleClick() {
		count += 1;
	}
</script>
```
