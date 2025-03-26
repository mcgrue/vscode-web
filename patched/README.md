(This is a "fork" of https://github.com/microsoft/vscode)

### index.html
* based on `src/vs/code/browser/workbench/workbench.html`
* uses "modules" instead of "node_modules" for `webPackagePaths`
	* if you were to deploy this to something like Cloudflare, they strip out "node_modules" because that's a common failure condition / "oopsie", but not for us!  We're doing it intentionally!
* add a *synchronous* fetch of workbench.json and set as value to meta tag with id `vscode-workbench-web-configuration`
	* this lets stock workbench.ts pick it up as workbench config
		* `workbench.json` isn't a "real" file in the vscode ecosystem - it's configuration data that MS uses to do "Product Configuration" that has stuff like "what's the name of the app?"
		* This data is inline and baked into individual products based off of the codebase
		* `workbench.json` is our encapsualtion of this Project Configuration Data

### extensionHostWorker.ts
* remove vscode's blocking of `postMessage` and `addEventListener`
	* lets extension send MessageChannel port up to host page
	* the microsoft version removes them for security reasons - someone could put malware into an extension and whoops - you get suborned!  But hey, 1) you control the extensions here and 2) it's a web-only deploy so any extension using nodejs will inherantly not work so - we unerfed them.  USE WITH CAUTION

### webWorkerExtensionHostIframe.html
* This is the html that's loaded in the iframe that's used as the container for all vscode extensions!
* add support for a `_port` message
	* lets extension send MessageChannel port up to host page
* remove hostname validation
  * not sure how this works but seems like it wouldn't
		* perhaps it could be left in...
* NOTE: any changes to script requires recomputing the integrity hash of script-src on CSP
	* copy contents of innerHTML of script (including whitespace up to tags) to ./script.txt
	* run `openssl dgst -sha256 -binary ./script.txt | openssl base64 -A` to get value after `sha256-`
