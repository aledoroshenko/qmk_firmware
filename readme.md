# Readme

I modified the Moonlander layout and stored it in a QMK repo since Oryx tool doesn’t support some useful features. To
use my modifications, I download the source code from Oryx, replace the files in the layout folder, compile it, and
flash it to Moonlander. Relevant files are in `/keyboards/moonlander/keymaps/oryxbranch`. I compile
using `qmk compile -kb moonlander -km oryxbranch`. The custom code is in `language-on-layer-switch` branch.

## Steps

1. Make updates to the layout in Oryx (easy way)
2. Make updates to the layout in this repo (hard way for custom features not available in Oryx)
3. Download source files from Oryx
4. Copy-paste it here and resolve conflicts
5. Commit changes
6. Compile layout
7. Flash layout to Moonlander



