# MapToolAddOnTemplate

Basic template for a MapTool Add-On Library with Powershell build script

## How To Use

- Clone repository OR 'Download as zip'
  - If cloned, you should set your own upstream repo, so changes aren't pushed to this template.
- Edit `library.json` to fill in your libary's info.
- Add scripts to the `library` directory.
- Create sub-directories for any web-apps.
- Run `build.ps1` when ready to deploy (see below).

## How to build and deploy your library

Information needed to build the library is stored in `buildinfo.json` and `.buildlocal`.
`.buildlocal` is intentinally not source-controlled and is where setting specific to your local environment are kept.

The first time `build.ps1` is run, it will prompt you for any missing info from `buildinfo.json` and `.buildlocal`.

The build script will iterate through any specified web-apps and execute `npm run pack`.

## How To create and package Overlays, Dialogs, and Frames

Create each overlay, dialog, or frame as a __node.js__ project in its own directory.
If you are experienced with webpack and code-splitting, you may want to bundle things differently,
but keeping everything isolated is easier to start with.

The build script will call `npm run pack` for each app.
This should be setup in the `package.json` to build the app and pack it into the `/dist` directory.
From what I can tell, MapTool supports up to ES5 (at leas, it does not support all of ES6), so make sure the package and any transpilers don't target anything later than ES5.
I use __webpack__ to bundle the apps up, but that is not strictly necessary.

After bundling, the build script will copy the contents of the `/dist` directory to the `library/public/<app-name-here>` directory before zipping up the entire library.

### Example

The following is an example of creating a web-app for a HTML5 Frame in MapTool.
Prerequisites: Have __Node.js__ and __npm__ installed
_For this example, the library's namespace is assumed to be `mysample.addon`._

- Create a folder named `sheet` in the project's root directory.
- In the `sheet` directory run `npm install webpack webpack-cli`
  - For details on how to configure __webpack__, please see [webpack's documentation](https://webpack.js.org/concepts/).
  - The build script assumes that the output is the contents of the `/dist` directory.
- Open the generated `package.json` file and ensure the following scripts are included:
  - Note the `build` script isn't strictly neccessary, but it helps when developing and debugging with a browser.

```json
{
  ...other settings here...
  "scripts": {
    "build": "webpack",
    "pack": "webpack --mode=production"
  }
  ...other settings here...
}
```

- Create a MT Script in `/library/mtscript/public` called `showCharacterSheet.mts`.
- Set the contents of this file to `[h: html.frame5("Character Sheet", "lib://mysample.addon/sheet/index.html")]`
- After writing your HTML5 application, run the `build.ps1` script.
- Add the `.mtlib` file to a campaign.
- In MapTool, show the frame by typing `[macro("showCharacterSheet@lib:mysample.addon"):'']` in the chat.
  - This can, and probably should be, called in other ways as well, like in an `onInit` event handler and a Campaign macro button.

## Miscelaneous

You should probably add any web-apps library directories to you .gitignore.
For the above example that would be adding `library/public/sheet` the the .gitignore.
