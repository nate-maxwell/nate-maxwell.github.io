---
layout: post
title: "[UE] Shelves"
date: 2024-04-04
permalink: /ue-shelves/
author: Nate Maxwell
categories:
    - "Unreal"
tags:
    - "unreal"
    - "python"
    - "ui"
    - "ux"
    - "umg"
---


I've had a shift in how I do editor toolbars in unreal since UE5. Previously I
would create a toolbar in UMG and load the asset on startup in my
`init_unreal.py` file from my plugin like so:

```python
from pathlib import Path

import unreal

def launch_toolbar() -> None:
    toolbar_path = Path('/PluginName/UI/plugin_toolbar.plugin_toolbar')
    pipeline_toolbar = unreal.load_asset(toolbar_path.as_posix())
    util_subsys = unreal.get_editor_subsystem(unreal.EditorUtilitySubsystem)
    util_subsys.spawn_and_register_tab(pipeline_toolbar)

launch_toolbar()
```

Side note - to get asset paths from the primary project content folder, you
would use a path like `/Game/dir/dir/AssetName.AssetName`, but for plugins the
`/Game/` directory is instead the plugin name, and there is no substitute for
the 'content' folder - `/PluginName/dir/dir/AssetName.AssetName`.

I really dislike the number of path types in unreal, which are shared by both
editor utility blueprints and the python API:

* Display Name & Asset Name = `AssetName`
* Path Name & Object Path   = `/Game/dir/dir/AssetName.AssetName`
* Package Path              = `/Game/dir/dir`
* Package Name              = `/Game/dir/dir/AssetName`

Seriously, why are there so many?!

Here you can see the UMG toolbar:

<p align="center">
<img src="https://i.imgur.com/HaFm5S6.png">
</p>

This has been great for doing blueprint startup scripts without needing to load
another asset, but lately I've been doing the opposite. I use this previously
described method for instantiating classes that have blueprint only logic on
construct but have been using the following python code to add a menu button
directory to the editor.

```python
import unreal

def create_button(section_name: str,
                  command: str,
                  name: str,
                  small_style_name: unreal.Name) -> None:
    """
    Creates a button on the editor play toolbar from the given args. The button command is created
    in the caller function.

    Args:
        section_name(str): The toolbar section name to place the button in.
        command(Str): The python command for the button to execute.
         This usually should import and call the main() func of a module.
        name(str): The name of the button.
        small_style_name(unreal.Name): The name of the icon to use  for the
         button. Icon names can be found at:
         https://github.com/EpicKiwi/unreal-engine-editor-icons
    """
    tool_menu = unreal.ToolMenus.get()
    level_menu_bar = tool_menu.find_menu(
        unreal.Name('LevelEditor.LevelEditorToolBar.PlayToolbar')
    )
    level_menu_bar.add_section(
        section_name=unreal.Name(section_name),
        label=unreal.Text(section_name)
    )

    entry = unreal.ToolMenuEntry(type=unreal.MultiBlockType.TOOL_BAR_BUTTON)
    entry.set_label(unreal.Text(name))
    entry.set_tool_tip(unreal.Text(name))
    entry.set_icon(unreal.Name('EditorStyle'), small_style_name)
    entry.set_string_command(
        type=unreal.ToolMenuStringCommandType.PYTHON,
        custom_type=unreal.Name(''),
        string=command
    )
    level_menu_bar.add_menu_entry(unreal.Name(section_name), entry)
    tool_menu.refresh_all_widgets()

def create_asset_importer_button() -> None:
    """Creates the asset importer button."""
    command = (
        'from lucid.unreal.asset_browser import main;'
        'global editor;'
        'editor = main()'
    )
    create_button('Lucid', command, 'Asset Browser',
                  unreal.Name('InputBindingEditor.MainFrame'))

def main() -> None:
    create_asset_importer_button()
    # and other buttons...
```

Which results in much more screen space for artists like so:

<p align="center">
<img src="https://i.imgur.com/u3DI3rY.png">
</p>

Either method works, but I feel like I have much more control in the UMG method
for generating layout and stylesheets, although my current UMG toolbar buttons
are rather large.
