# Readme

Repo with custom Moonlander layouts with updates on top of Oryx-generated layouts, 
which are not supported by Oryx natively (set layer to base, react on layer changes).

## Workflow

1. Update layout in Oryx
2. Make updates to the layout in this repo (hard way for custom features not available in Oryx)
3. Download source files from Oryx
4. Copy-paste it in the folder with layout (`moonlander/keymaps/udon`), overwrite all
5. In diff check and bring back needed changes (reset to a base layer, function to switch language on layer change)
5. Commit changes
6. Compile layout (`qmk compile -kb moonlander -km udon`, `qmk clean -a` if some issues)
7. Flash layout to Moonlander
