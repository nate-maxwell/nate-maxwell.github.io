---
layout: post
title: "[UE] Plugin Manager"
date: 2024-08-11
permalink: /ue-plugin-manager/
author: Nate Maxwell
categories:
    - "Unreal"
    - "Plugins"
    - "Python"
---

I've been poking around the broader facility pipeline code and see this
interesting implementation. Each python module has a `/module/src/code.py` file,
but there is also a `/module/ctx/context.py` file in every repo. This
`context.py` file gets loaded by the sys path manager when the module is
imported at runtime, and it always contains a context class that describes
implementation and tracking details, namely in the form of relating environment
variables to module variables.

This got me thinking about our unreal setup in the Viz department. We work
heavily with plugins as we can have anywhere from 2 to 4 projects running at
one time, and they could last anywhere from 2 days to 18 months. Plugins make
it extremely easy to get a new project up and running.

I've recently overhauled our unreal toolbar to create buttons in the play
toolbar rather than using a widget the user must insert at the top of the
window.

Each plugin now contains a `context.json` file that we've started using for
broader application understanding of the currently loaded toolset. The file is
located in the plugin root. A simplistic version would look like this:

```json
{
	"plugin_name": "Example Toolset",
	"startup_cmd": "import example_toolset; example_toolset.main()"
}
```

Additionally, each plugin contains a simple python file that runs the primary
entry point for the plugin, typically a widget, and would look something like
this:

```python
# example_toolset.py

from pathlib import Path

import unreal

_UTIL_SUBSYS = unreal.get_editor_subsystem(unreal.EditorUtilitySubsystem)

PLUGIN_NAME = 'Example_Toolset'

def launch_plugin_menu() -> None:
    menu_path = Path(f'/{PLUGIN_NAME}/PluginMenu.PluginMenu')
    widget = unreal.load_asset(menu_path.as_posix())
    _UTIL_SUBSYS.spawn_and_register_tab(widget)

def main() -> None:
    launch_plugin_menu()

if __name__ == '__main__':
    main()
```

Since these are in the plugin's `Content/Plugins` directory, there's no need to
worry about managing them on the path.

From here we call the `ue_plugins.py` file from our `init_unreal.py` file to
loop through all plugins and check for a context.json file.

```python
# ue_plugins.py

import json
from pathlib import Path

import editor

import unreal

PROJECT_DIR = Path(unreal.SystemLibrary.get_project_directory())
PLUGINS_DIR = Path(PROJECT_DIR, 'Plugins')

PLUGIN_NAME_K = 'plugin_name'
PLUGIN_CMD_K = 'startup_cmd'

SHELF_ICON = unreal.Name('EditorViewport.ShaderComplexityMode')
BUTTON_ICON = unreal.Name('WidgetDesigner.LayoutTransform')


def load_toolbar_plugins() -> None:
    menu = editor.create_toolbar_submenu(section_name='Lucid',
                                         dropdown_name='Plugins',
                                         small_style_name=SHELF_ICON)
    for p in PLUGINS_DIR.glob('*'):
        ctx_file = Path(p, 'context.json')
        if not ctx_file.exists():
            continue  # Not a pipeline friendly plugin

        with open(ctx_file) as file:
            ctx_data = json.load(file)
        label = ctx_data[PLUGIN_NAME_K]
        command = ctx_data[PLUGIN_CMD_K]

        editor.add_dropdown_button(menu, label, command, BUTTON_ICON)
```

with the `editor.create_toolbar_submenu` and `editor.add_dropdown_button`
functions coming from `editor.py` and are as follows:

```python
# editor.py

import unreal

_DEFAULT_ICON = unreal.Name('Log.TabIcon')
_SECTION = unreal.Name('actions')
_SECTION_LABEL = unreal.Text('Actions')

_OWNING_MENU = 'LevelEditor.LevelEditorToolBar.PlayToolBar'


def _get_play_toolbar(menu_name: str = _OWNING_MENU) -> unreal.ToolMenu:
    tool_menus = unreal.ToolMenus.get()
    return tool_menus.find_menu(
        unreal.Name(menu_name)
    )


def create_toolbar_submenu(section_name: str,
                           dropdown_name: str,
                           small_style_name: unreal.Name = _DEFAULT_ICON
                           ) -> unreal.Name:
    """Add a dropdown to the Play toolbar.

    Args:
        section_name (str): The toolbar section to group under (created if
         missing).
        dropdown_name (str): The visible name of the dropdown.
        small_style_name(unreal.Name): The name of the icon to use for the
         button.
    Returns:
        unreal.Name: The submenu id.
    """
    tool_menus = unreal.ToolMenus.get()
    play_toolbar = _get_play_toolbar()

    play_toolbar.add_section(
        section_name=unreal.Name(section_name),
        label=unreal.Text(section_name)
    )

    entry_name = unreal.Name(f'{dropdown_name.replace(" ", "")}')
    combo = unreal.ToolMenuEntry(
        name=entry_name,
        type=unreal.MultiBlockType.TOOL_BAR_COMBO_BUTTON
    )
    combo.set_label(unreal.Text(dropdown_name))
    combo.set_tool_tip(unreal.Text(dropdown_name))
    combo.set_icon(unreal.Name('EditorStyle'), small_style_name)
    play_toolbar.add_menu_entry(unreal.Name(section_name), combo)

    popup_id = unreal.Name(f'{_OWNING_MENU}.{entry_name}')
    popup = tool_menus.find_menu(popup_id) or tool_menus.register_menu(popup_id)
    popup.add_section(_SECTION, _SECTION_LABEL)

    tool_menus.refresh_all_widgets()
    return popup_id


def add_dropdown_button(menu_id: unreal.Name,
                        label: str,
                        command: str,
                        small_style_name: unreal.Name = _DEFAULT_ICON) -> None:
    """Add a menu item to an existing drop-down menu.
    Args:
        menu_id (unreal.Name): The submenu id to add to.
        label (str): The entry label.
        command (str): The string python command for the button to exec.
        small_style_name(unreal.Name): The name of the icon to use for the
         button.
    """
    tool_menus = unreal.ToolMenus.get()
    popup = tool_menus.find_menu(menu_id) or tool_menus.register_menu(menu_id)
    popup.add_section(_SECTION, _SECTION_LABEL)

    str_id = unreal.StringLibrary.conv_name_to_string(menu_id)
    entry = unreal.ToolMenuEntry(
        name=unreal.Name(
            # Ensure unique name
            f'Item_{hash((str_id, label)) & 0xffffffff:x}'),
        type=unreal.MultiBlockType.MENU_ENTRY
    )
    entry.set_label(unreal.Text(label))
    entry.set_tool_tip(unreal.Text(label))
    entry.set_string_command(
        type=unreal.ToolMenuStringCommandType.PYTHON,
        custom_type=unreal.Name(''),
        string=command
    )
    entry.set_icon(unreal.Name('EditorStyle'), small_style_name)

    popup.add_menu_entry(unreal.Name('actions'), entry)
    tool_menus.refresh_menu_widget(menu_id)
```

So we start up the engine, search all the plugins for a `context.json` file,
and then run the string python command to launch and register a plugin widget.
Our core plugin that manages the unreal toolbar is loaded into every project,
and from here we optionally load each other plugin and add a button to the
toolbar for them.

<img src="https://i.imgur.com/QT0oHvC.png">

This is a simplistic version of our implementation, as the `context.json` file
contains more data about what kind of tool is being inspected. Some plugins are
considered 'core' plugins and get treated special, while others are project
dependent and get added to the plugins drop down.

Maybe this isn't anything interesting to you, or maybe it inspires you to think
about your plugin management in Unreal. Either way I thought it was worth
sharing.
