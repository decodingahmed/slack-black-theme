# Slack Black Theme

A darker, more contrasty theme for Slack version 3. Slack v4 has a built-in dark theme.

# Installing into Slack

Find your Slack's application directory.

* Windows: `%homepath%\AppData\Local\slack\`
* Mac: `/Applications/Slack.app/Contents/`

Open up the most recent version (e.g. `app-3.3.7`) then open
`resources\app.asar.unpacked\src\static\ssb-interop.js`

At the very bottom of the file (not inside any brackets), add:

```js
// First make sure the wrapper app is loaded
document.addEventListener("DOMContentLoaded", function() {

  // Then get its webviews
  let webviews = document.querySelectorAll(".TeamView webview");

  // Fetch our CSS in parallel ahead of time
  const cssPath = 'https://raw.githubusercontent.com/ahmedrzaman/slack-black-theme/master/custom.css';
  let cssPromise = fetch(cssPath).then(response => response.text());

  let customCustomCSS = `
  :root {
     /* Modify these to change your theme colors: */
     --primary: #61AFEF;
     --text: #ABB2BF;
     --background: #222;
     --background-elevated: #222;
     --background-bright: #999;
     --scrollbar-background: var(--text);
     --scrollbar-border: var(--text);
     --text-bright: white;
  }
  `

  // Insert a style tag into the wrapper view
  cssPromise.then(css => {
     let s = document.createElement('style');
     s.type = 'text/css';
     s.innerHTML = css + customCustomCSS;
     document.head.appendChild(s);
  });

  // Wait for each webview to load
  webviews.forEach(webview => {
     webview.addEventListener('ipc-message', message => {
        if (message.channel == 'didFinishLoading')
           // Finally add the CSS into the webview
           cssPromise.then(css => {
              let script = `
                    let s = document.createElement('style');
                    s.type = 'text/css';
                    s.id = 'slack-custom-css';
                    s.innerHTML = \`${css + customCustomCSS}\`;
                    document.head.appendChild(s);
                    `
              webview.executeJavaScript(script);
           })
     });
  });
});
```

Restart Slack.

NB: You'll have to do this every time Slack updates.


# Development

`git clone` the project and `cd` into it.

Change the CSS URL to `const cssPath = 'http://localhost:8080/custom.css';`

Run a static webserver of some sort on port 8080:

```bash
npm install -g static
static .
```

In addition to running the required modifications, you will likely want to add auto-reloading:

```js
const cssPath = 'http://localhost:8080/custom.css';
const localCssPath = '/Users/bryankeller/Code/slack-black-theme/custom.css';

window.reloadCss = function() {
   const webviews = document.querySelectorAll(".TeamView webview");
   fetch(cssPath + '?zz=' + Date.now(), {cache: "no-store"}) // qs hack to prevent cache
      .then(response => response.text())
      .then(css => {
         console.log(css.slice(0,50));
         webviews.forEach(webview =>
            webview.executeJavaScript(`
               (function() {
                  let styleElement = document.querySelector('style#slack-custom-css');
                  styleElement.innerHTML = \`${css}\`;
               })();
            `)
         )
      });
};

fs.watchFile(localCssPath, reloadCss);
```

Instead of launching Slack normally, you'll need to enable developer mode to be able to inspect things.

* Mac: `export SLACK_DEVELOPER_MENU=true; open -a /Applications/Slack.app`

* Windows: (todo)

# License

Apache 2.0
