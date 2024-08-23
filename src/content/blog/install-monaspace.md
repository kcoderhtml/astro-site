---
title: Monaspace VS-Code install
description: Want to use the cool Monaspace font with fancy ligatures in VS Code? I got you! This guide walks you through downloading the font, configuring VS Code with a custom CSS file, and even includes some tips for finding the right class names for different VS Code elements. Enjoy your beautiful new coding setup!
slug: monaspace-vs-code-install
date: 2023-11-10
---

# Monaspace VS-Code install

![monaspace font in action](https://assets.vrite.io/64974cb888e8beebeb2c925b/KuOAwCEm9ypWEemv60Qs7.png)

To install the Monaspace font on macOS with VS Code and enable multifont syntax highlighting with the CSS JS Loader extension, you can follow these steps:

1.  Download and install the Monaspace font:

    - Visit [https://github.com/githubnext/monaspace/releases/latest](https://github.com/githubnext/monaspace/releases/latest) and download the zip.
    - Install the Monaspace font:

      - On macOS, drag the font files into font book.
      - For windows, drag into the font window in settings.
      - For Linux, clone the repo and run:
        ```bash
        cd util
        ./install_linux.sh
        ```

2.  Configure VS Code

    - Install the [Custom CSS and JS Loader](https://marketplace.visualstudio.com/items?itemName=be5invis.vscode-custom-css) plugin.
    - Set font to one of the below options: `Monaspace Neon Var`, `Monaspace Argon Var`, `Monaspace Xeon Var`, `Monaspace Radon Var`, or `Monaspace Krypton Var`.

      - You will find this option under _Editor: Font Family_ in the user preferences

        ![available fonts](https://assets.vrite.io/64974cb888e8beebeb2c925b/v0cMm5jcwHEgrvtBv4Syx.png)

    - Enable font ligatures in the settings.json with below snippet:
      ```json
      "editor.fontLigatures": "'ss01', 'ss02', 'ss03', 'ss04', 'ss05', 'ss06', 'ss07', 'ss08', calt', 'dlig'",
      ```
    - Enable the custom CSS file with this in the `settings.json` modifying the file path for Windows / MacOS / Linux if needed:
      ```json
          "vscode_custom_css.imports": [
              "file:///Users/{{user}}/.vscode/style.css", // for mac (remove if not mac)
              "file://\C\Users\{{user}}\vscode/style.css" // for windows (remove if not windows)
              "file:///home/{{user}}/.vscode/style.css" // for linux (remove if not windows)
          ],
      ```

3.  Create custom CSS file at the path you specified above.

    - Depending on your VS Code version, the class names might be different, so you may need to use the developer tools to find the correct one.
    - The code that work for me on `VS Code version: 1.84.2 (Universal) commit: 1a5daa3a0231a0fbba4f14db7ec463cf99d7768e` are here:

      ```css
      /* Comment Class */
      .mtk3 {
        font-family: "Monaspace Radon Var";
        font-weight: 500;
      }

      /* Copilot Classes */
      .ghost-text-decoration {
        font-family: "Monaspace Krypton Var";
        font-weight: 200;
      }

      .ghost-text-decoration-preview {
        font-family: "Monaspace Krypton Var";
        font-weight: 200;
      }
      ```

    - Thanks to **[@fspoettel](https://github.com/fspoettel)** on GitHub for this trick to get the copilot classes when in dev mode:

      - You can inspect transient DOM elements by halting the app with a `debugger` after a delay with a console statement like:
        ```ts
        setTimeout(() => {
          debugger;
        }, 10000);
        ```

    - Before you are finished, you have to run `Enable Custom CSS and JS` from the command bar.

That should be it! Hopefully you will have a beautiful custom font VS Code install.

If you are looking for a good theme, I can highly recommend the [Catppuccin](https://marketplace.visualstudio.com/items?itemName=Catppuccin.catppuccin-vsc) theme, as that is what I use myself. Be sure to check out [Monaspace’s webstite](https://monaspace.githubnext.com/) as it is a work of art. Happy Coding! 👩‍💻

*Updated 2024-08-22: changed mtk4 to mtk3 on the feedback of [mutammim](https://github.com/mutammim)*

