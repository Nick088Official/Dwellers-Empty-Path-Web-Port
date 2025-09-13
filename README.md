# Dweller's Empty Path - Web Port

A project for web porting the free RPG Maker MV game by **Temmie Chang**, Dweller's Empty Path.


## Usage

Go to https://nick088official.github.io/Dwellers-Empty-Path-Web-Port/www/


## How?

This guide provides step-by-step instructions for modifying the game to run correctly in a web browser. The process involves fixing asset pointers, patching a desktop-only plugin, and optionally creating mobile-compatible audio files.

### Prerequisites

1.  **Get the Game Files:** [Download the original game from the creator's itch.io page](https://tuyoki.itch.io/dwellers-empty-path) and extract the files.
2.  **Get the Decrypter (Optional, for mobile audio compatibility):** [Download RPGMakerDecrypter Latest Release Binaries](https://github.com/uuksu/RPGMakerDecrypter/releases/latest) (.exe for windows, the other cli file is for MacOS/Linux)
3.  **A Text Editor:** To modify files (e.g., VS Code, Sublime Text, Notepad).
4.  **Python (for testing):** Having [Python](https://www.python.org/downloads/) installed is the easiest way to run a local server to test the game.
5.  **FFmpeg (Optional, for mobile audio compatibility):** For the mobile audio fix, you will need [FFmpeg](https://ffmpeg.org/download.html) installed and added to your system's PATH.

### Part A: Decryption (Optional, for mobile audio compatibility)

As the game's `audio` and `img` folders are encrypted, it uses certain file extensions:
- `.rpgmvo` for desktop audio (encrypted .ogg)
- `.rpgmvm` for mobile audio (encrypted .m4a)
- `.rpgmvp` for images (encrypted .png)

RPG Maker MV uses 2 different audio formats for mobile and desktop, because the mobile one is more size efficient. Since it wasn't supposed to run on mobile, it doesn't contain `.rpgmvm` files, meaning it wouldn't play any audio on mobile.


To Decrypt the files, open the CMD/Terminal in the folder containing the RPGMakerDecrypter Binaries:
- For Windows:
    ```cmd
    RPGMakerDecrypter-cli.exe <Your-Game-Folder-Directory>
    ```
- For macOS & Linux:
    ```bash
    RPGMakerDecrypter-cli <Your-Game-Folder-Directory>
    ```

Now it contains both encrypted and decrypted files, following on it will be explained how to clean up and fix the mobile audio issue.

### Part B: File Cleanup

For a clean web game, you only need the `www` folder.

1.  Delete everything else except the `www` folder in the game files.
2.  Inside the `www` folder, you can safely delete the `package.json` file.
3. Open the CMD/Terminal in the `www` folder and run the following commands to get rid of encrypted files:
- For Windows:
    ```cmd
    del /S /Q "audio\*.rpgmvo" && del /S /Q "img\*.rpgmvp"
    ```
- For macOS & Linux:
    ```bash
    find audio img -type f \( -name "*.rpgmvo" -o -name "*.rpgmvp" \) -delete
    ```

### Part B: Core Browser Compatibility Fixes

These steps are **mandatory** to get the game to run at all in a browser.

#### Patch the Bitmap Font Plugin

The original `Bitmap Fonts.js` plugin uses desktop-only commands (`require`) to scan your file system. This will crash the game instantly in a browser. It must be replaced with a web-compatible version.

1.  Navigate to the `www/js/plugins/` folder and open **`Bitmap Fonts.js`**.
2.  **Delete all the content** in the file.
3.  Replace it by pasting in the complete, web-patched code that uses web requests instead of file-system commands:
    ```js
    //=============================================================================
    // TDS Bitmap Fonts
    // Version: 1.6 (Web-Patched)
    //=============================================================================
    // Add to Imported List
    var Imported = Imported || {} ; Imported.TDS_BitmapFonts = true;
    // Initialize Alias Object
    var _TDS_ = _TDS_ || {} ; _TDS_.BitmapFonts = _TDS_.BitmapFonts || {};
    //=============================================================================
    /*:
    * @plugindesc
    * This plugin allows you to create and set images as fonts.
    * This plugin is made exclusively for Archeia and her projects.
    *
    * @author TDS
    *
    * @help
    * ============================================================================
    * * Text Codes
    * ============================================================================
    *
    *  This text code will allow you to set the bitmap font tone for
    *  messages:
    *
    *    \BC[red, green, blue]
    *
    *     (Red, Green, and Blue are values of strength from -255 to 255)
    *
    *    Example:
    *
    *    \BC[255,0,0]Show me some RED!\BC[]And Now back to normal.
    */
    //=============================================================================
    function BitmapFontManager(){throw new Error("This is a static class")}BitmapFontManager._fonts={},BitmapFontManager._isLoading=!1,BitmapFontManager._queue=[],BitmapFontManager._totalFiles=0,BitmapFontManager._loadedFiles=0,BitmapFontManager.isReady=function(){return!this._isLoading},BitmapFontManager.doesBitmapFontExist=function(t){return!!this._fonts[t]},BitmapFontManager.getFontData=function(t){return this._fonts[t]},BitmapFontManager.hexToRGB=function(t){return t=t.replace("#",""),[parseInt(t.substring(0,2),16),parseInt(t.substring(2,4),16),parseInt(t.substring(4,6),16)]},BitmapFontManager.processAtlasData=function(t){for(var a={characters:{},bitmapName:t.meta.image.slice(0,-4)},e={_scx0:" ",_scx1:"\\",_scx2:"/",_scx3:":",_scx4:"*",_scx5:"?",_scx6:"<",_scx7:">",_scx8:"|",_scx9:".",_scx10:'"'},i=Object.keys(t.frames),r=0;r<i.length;r++){var o=i[r].slice(0,-4),s=t.frames[i[r]];e[o]?o=e[o]:2===o.length&&(o=o[0].toLowerCase()),a.characters[o]={},a.characters[o].rect=s.frame,a.characters[o].originalRect=s.spriteSourceSize,a.characters[o].sourceSize=s.sourceSize}return a},BitmapFontManager.makeBitmapFontObject=function(t,a,e){e=e||[];var i={name:t,size:a,color:e},r=this._fonts[t];if(r&&r.settings)for(var o=Object.keys(r.settings.fonts),s=0;s<o.length;s++){var n=o[s],l=r.settings.fonts[n],h=l.fontRange,c=l.forceTone;if(a>=h[0]&&a<=h[1]){if(i.atlas=r.atlases[n],e.length>0){var g=ImageManager.loadBitmapFontImage(t,i.atlas.bitmapName),p=new Bitmap(g.width,g.height);p.blt(g,0,0,g.width,g.height,0,0),c?p.forceTone(e[0],e[1],e[2]):p.adjustTone(e[0],e[1],e[2]),i.bitmap=p}else i.bitmap=ImageManager.loadBitmapFontImage(t,i.atlas.bitmapName);i.fontHeight=l.fontHeight,i.spaceWidth=l.spaceWidth;break}}return i},BitmapFontManager.loadAllBitmapFonts=function(){this._isLoading=!0,this._queue=["GameFont"],this.loadNextFontFromQueue()},BitmapFontManager.loadNextFontFromQueue=function(){if(0===this._queue.length)return void(this._isLoading=!1);var t=this._queue.shift(),a="fonts/Bitmap Fonts/"+t+"/Settings.json";this.loadJsonFile(t,"settings",a,this.onSettingsFileLoaded.bind(this))},BitmapFontManager.onSettingsFileLoaded=function(t,a,e){if(!e||!e.fonts)return console.error('Bitmap Fonts Error: Settings.json for font "'+t+'" is missing or malformed.'),void this.loadNextFontFromQueue();this._fonts[t]||(this._fonts[t]={settings:null,atlases:{}});var i=Object.keys(e.fonts);if(this._fonts[t].settings=e,this._totalFiles+=i.length,0===i.length)this.loadNextFontFromQueue();else for(var r=0;r<i.length;r++){var o=i[r];ImageManager.loadBitmapFontImage(t,o,0);var s="fonts/Bitmap Fonts/"+t+"/"+o+".json";this.loadJsonFile(t,o,s,this.onAtlasFileLoaded.bind(this))}},BitmapFontManager.onAtlasFileLoaded=function(t,a,e){this._fonts[t].atlases[a]=this.processAtlasData(e),this._loadedFiles++,this._loadedFiles>=this._totalFiles&&this.loadNextFontFromQueue()},BitmapFontManager.loadJsonFile=function(t,a,e,i){var r=new XMLHttpRequest;r.open("GET",e),r.overrideMimeType("application/json"),r.onload=function(){404===r.status?i(t,a,null):r.status<400?i(t,a,JSON.parse(r.responseText)):(console.error("Failed to load Bitmap Font file: "+e),i(t,a,null))},r.onerror=function(){console.error("Failed to load Bitmap Font file: "+e),i(t,a,null)},r.send()},ImageManager.loadBitmapFontImage=function(t,a,e){return this.loadBitmap("fonts/Bitmap Fonts/"+t+"/",a,e,!1)},_TDS_.BitmapFonts.Scene_Boot_initialize=Scene_Boot.prototype.initialize,Scene_Boot.prototype.initialize=function(){_TDS_.BitmapFonts.Scene_Boot_initialize.call(this),BitmapFontManager.loadAllBitmapFonts()},_TDS_.BitmapFonts.Scene_Boot_isReady=Scene_Boot.prototype.isReady,Scene_Boot.prototype.isReady=function(){return!!_TDS_.BitmapFonts.Scene_Boot_isReady.call(this)&&BitmapFontManager.isReady()},_TDS_.BitmapFonts.Bitmap_initialize=Bitmap.prototype.initialize,_TDS_.BitmapFonts.Bitmap_drawText=Bitmap.prototype.drawText,_TDS_.BitmapFonts.Bitmap_measureTextWidth=Bitmap.prototype.measureTextWidth,Bitmap.prototype.initialize=function(t,a){_TDS_.BitmapFonts.Bitmap_initialize.call(this,t,a),this._bitmapFont=null,this._useBitmapFont=!0,this._bitmapFontColor=null},Object.defineProperty(Bitmap.prototype,"bitmapFontColor",{get:function(){return this._bitmapFontColor},set:function(t){Array.isArray(t)?t.equals(this._bitmapFontColor)||(this._bitmapFontColor=t,this.updateBitmapFont()):this._bitmapFontColor!==t&&(this._bitmapFontColor=t,this.updateBitmapFont())},configurable:!0}),Bitmap.prototype.isUsingBitmapFont=function(){return this._useBitmapFont},Bitmap.prototype.measureTextWidth=function(t,a=!1){return this.isUsingBitmapFont()?this.measureBitmapFontText(t,a).width:_TDS_.BitmapFonts.Bitmap_measureTextWidth.call(this,t)},Bitmap.prototype.measureBitmapFontText=function(t,a=!1){a&&this.updateBitmapFont();var e={width:0,height:0},i=this._bitmapFont;if(i&&void 0!==t){e.height=i.fontHeight;for(var r=t.toString().split(""),o=0;o<r.length;o++){var s=i.atlas.characters[r[o]];e.width+=s?s.rect.w:i.spaceWidth}}return e},Bitmap.prototype.drawText=function(t,a,e,i,r,o){this.isUsingBitmapFont()?this.drawBitmapFontText(t,a,e,i,r,o):_TDS_.BitmapFonts.Bitmap_drawText.call(this,t,a,e,i,r,o)},Bitmap.prototype.updateBitmapFont=function(){if(BitmapFontManager.doesBitmapFontExist(this.fontFace))if(this._bitmapFont){var t=this._bitmapFont;t.name===this.fontFace&&t.size===this.fontSize&&t.color.equals(this._bitmapFontColor)||(this._bitmapFont=BitmapFontManager.makeBitmapFontObject(this.fontFace,this.fontSize,this._bitmapFontColor))}else this._bitmapFont=BitmapFontManager.makeBitmapFontObject(this.fontFace,this.fontSize,this._bitmapFontColor);else this._bitmapFont=null},Bitmap.prototype.drawBitmapFontText=function(t,a,e,i,r,o){this.updateBitmapFont();var s=this._bitmapFont;if(s&&void 0!==t){var n=s.bitmap,l=a,h=e+r-(r-.7*s.fontHeight)/2;if("center"===o)l+=(i-this.measureTextWidth(t))/2;if("right"===o)l+=i-this.measureTextWidth(t);for(var c=t.toString().split(""),g=0,p=0;p<c.length;p++){var B=c[p],F=s.atlas.characters[B];if(F){var _=F.sourceSize.h-s.fontHeight,f=F.rect,d=(r-s.fontHeight)/4,u=h-f.h+_+d;this.blt(n,f.x,f.y,f.w,f.h,l+g,u),g+=f.w}else g+=s.spaceWidth}}},Bitmap.prototype.forceTone=function(t,a,e){if((t||a||e)&&this.width>0&&this.height>0){for(var i=this._context,r=i.getImageData(0,0,this.width,this.height),o=r.data,s=0;s<o.length;s+=4)o[s+0]=t,o[s+1]=a,o[s+2]=e;i.putImageData(r,0,0),this._setDirty()}},_TDS_.BitmapFonts.Window_Base_resetFontSettings=Window_Base.prototype.resetFontSettings,_TDS_.BitmapFonts.Window_Base_processEscapeCharacter=Window_Base.prototype.processEscapeCharacter,Window_Base.prototype.resetFontSettings=function(){_TDS_.BitmapFonts.Window_Base_resetFontSettings.call(this),this.contents.updateBitmapFont()},Window_Base.prototype.obtainMultiEscapeParam=function(t){var a=/^\[([^\]]*)\]/.exec(t.text.slice(t.index)),e=[];return a&&(t.index+=a[0].length,e=eval(a[0])),e},Window_Base.prototype.processEscapeCharacter=function(t,a){switch(t){case"BC":var e=this.obtainMultiEscapeParam(a);this.contents.bitmapFontColor=this.obtainBitmapFontColor(e)}_TDS_.BitmapFonts.Window_Base_processEscapeCharacter.call(this,t,a)},Window_Base.prototype.obtainBitmapFontColor=function(t){return t.length>0?1===t.length?BitmapFontManager.hexToRGB(this.textColor(t[0])):t:null},Window_Base.prototype.processNormalCharacter=function(t){var a=t.text[t.index++],e=this.textWidth(a);this.contents.drawText(a,t.x,t.y,2*e,t.height),t.x+=e};
    ```
4. Save the file.

#### Patch the System Encrypted Assets Usage (Optional, for mobile audio compatibility)

The game uses Encrypted Assets, `audio` and `img` folders, but those are decrypted for mobile audio compatibility.

1.  With a text editor, open the `System.json` file in the `data` folder inside the`www` folder.
2. Search for `hasEncryptedImages` and `hasEncryptedAudio`, you could use your text editor search option if it has one.
3. Change their values from `true` to `false`.
4. Save the file.

### Part C: Mobile Audio Compatibility Fix (Optional, for mobile audio compatibility)

On mobile, the game tries to load `.m4a` audio files by default, but the decrypted game files contain only contains `.ogg` files, causing a crash. To fix this for maximum compatibility, you can generate `.m4a` copies.

1.  Ensure you have **FFmpeg installed and added to your system's PATH**.
2.  Open a CMD/Terminal in your **root game folder** (the one containing the `www` folder).
3.  Paste and run the following one-liner command. It will find every `.ogg` file and create an `.m4a` copy next to it:
    - For Windows:
        ```cmd
        for /R "www\audio" %F in (*.ogg) do ffmpeg -i "%F" -v quiet -stats "%~dpnF.m4a"
        ```
    - For macOS & Linux:
        ```bash
        find www/audio -type f -name "*.ogg" -exec sh -c 'ffmpeg -i "$0" -v quiet -stats "${0%.ogg}.m4a"' {} \;
        ```

### Part D: Add Responsive Scaling (Fix Zoom & Scrolling)

By default, the game canvas is a fixed size, causing scrollbars on small windows and incorrect zooming on mobile. This fix makes the game scale to fit any screen perfectly.

#### Step 1: Create the Stylesheet

1.  In your `www` folder, create a new folder named `css`.
2.  Inside `www/css`, create a new file named `style.css`.
3.  Paste the following code into `style.css`:
    ```css
    /* --- Core Page Styling --- */
    html, body {
        /* Set a black background to prevent white flashes */
        background-color: black;
        
        /* Remove default browser margins and padding */
        margin: 0;
        padding: 0;

        /* Disable all scrollbars */
        overflow: hidden;

        /* Make the body fill the entire screen */
        width: 100%;
        height: 100%;
    }

    /* --- Game Canvas Styling --- */
    canvas {
        /* Use absolute positioning to remove the canvas from the document flow */
        position: absolute;
        top: 0;
        left: 0;
        
        /*
        Explicitly tell the canvas to fill its container (the window),
        using !important to override any other styles
        */
        width: 100% !important;
        height: 100% !important;

        /* 
        It scales the canvas to fit within its container (the 100% width/height)
        while PRESERVING its original aspect ratio. This automatically creates
        black bars (letterboxing) if the window shape doesn't match the game's
        shape. While using !important to override any other styles
        */
        object-fit: contain !important;
    }
    ```

#### Step 2: Modify the HTML File

You need to add two lines to your `index.html` file to link the new stylesheet and ensure proper scaling.

1.  Open `www/index.html` in your text editor.
2.  Find the closing **`</head>`** tag.
3.  Paste the following 3 lines:
- Before the `</head>` tag, Replace the previous `<meta name="viewport"...` with:
    ```html
    <!-- This tag ensures the game scales correctly on mobile devices -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    ```
- Before the `</head>` tag, Add the following line before the `<title>` tag:
    ```html
    <!-- This links to your new stylesheet for responsive scaling -->
    <link rel="stylesheet" type="text/css" href="css/style.css">
    ```
- Add the following line before the `</body>` tag:
    ```html
    <!-- 
    It enables the engine's native stretch-to-fit
    mode and then immediately calls the engine's own resize function to force
    a correct layout calculation. This harmonizes scaling and touch input.
    -->
    <script>
        (function() {
            var _alias_scene_boot_start = Scene_Boot.prototype.start;
            Scene_Boot.prototype.start = function() {
                _alias_scene_boot_start.call(this);
                // Force the engine's stretch mode to be active
                Graphics._stretchEnabled = true;
                // Manually trigger the engine's own window resize logic
                Graphics._onWindowResize();
            };
        })();
    </script>
    ```

### Part E: Testing the Game Locally

Once you have made the modifications, you need to test them using a local web server.

1.  Open a terminal or command prompt **inside the root game folder**.
2.  Run the following command to start a simple web server:
    ```bash
    python -m http.server
    ```
3.  Open your web browser and go to the following address:
    **`http://localhost:8000/www/`**

The game should now start and be fully playable on both desktop and mobile (if you have done the optional mobile audio compatibility fixes).

### Part F: Bypass Jekyll GitHub Pages for Underscore Files (Optional)

This is only for people who are going to use GitHub Pages, which automatically uses Jekyll to build static blog websites. The issue is that Jekyll ignores files that start with '_', which some of the game assets do, so we will not use Jekyll.

1. In the root project folder, add an emtpy file named `.nojekyll`.


## Why?

For pure educational purposes, fun, and to make this game accessible on more platforms.


## Credits
- **Game:** All credit for the game's creation goes to **Temmie Chang**. Please support the original creator by visiting the [official itch.io page](https://tuyoki.itch.io/dwellers-empty-path).
- **Web Port:** This web-based version was ported by [Nick088](https://linktr.ee/nick088).
- **Game Engine:** RPG Maker MV.