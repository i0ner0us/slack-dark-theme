# Slack Dark Theme

Orginal Credit: https://github.com/widget-/slack-black-theme

### Version 3.3.8
Find your Slack's application directory.

* Windows: `%homepath%\AppData\Local\slack\`
* Mac: `/Applications/Slack.app/Contents/`
* Linux: `/usr/lib/slack/` (Debian-based)


Open up the most recent version (e.g. `app-3.3.8`) then open the following files:
`resources\app.asar.unpacked\src\static\index.js`  
`resources\app.asar.unpacked\src\static\ssb-interop.js`

At the very bottom, add

```js

document.addEventListener("DOMContentLoaded", function() {

   // Then get its webviews
   let webviews = document.querySelectorAll(".TeamView webview");

   // Fetch our CSS in parallel ahead of time
   const cssPath = 'https://cdn.rawgit.com/laCour/slack-night-mode/master/css/raw/black.css';
   let cssPromise = fetch(cssPath).then(response => response.text());

   let customCustomCSS = `
   :root {
      /* Modify these to change your theme colors: */
      --primary: #61AFEF;
      --text: #00b200;
  }
  div.c-message.c-message--light.c-message--hover
  {
    color: #00b200 !important;
  }

	span.c-message__body,
	a.c-message__sender_link,
	span.c-message_attachment__media_trigger.c-message_attachment__media_trigger--caption,
	div.p-message_pane__foreword__description span
	{
			color: #00b200 !important;
      font-family: "Fira Code", Arial, Helvetica, sans-serif;
      text-rendering: optimizeLegibility;
      font-size: 14px;
      letter-spacing: -0.6px !important;
	}
  div.c-virtual_list__scroll_container {
    background-color: #080808 !important;
}
.p-message_pane .c-message_list:not(.c-virtual_list--scrollbar), .p-message_pane .c-message_list.c-virtual_list--scrollbar > .c-scrollbar__hider {
     z-index: 0;
}
div.c-message__content:hover {
    background-color: #080808 !important;
}

div.c-message:hover {
    background-color: #080808 !important;
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

Notice that you can edit any of the theme colors using the custom CSS (for the already-custom theme.) Also, you can put any CSS URL you want here, so you don't necessarily need to create an entire fork to change some small styles.

That's it! Restart Slack and see how well it works.

NB: You'll have to do this every time Slack updates.
