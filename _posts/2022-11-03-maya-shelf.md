---
layout: post
title: "[MAYA] Custom Shelf"
date: 2022-11-03
permalink: /maya-custom-shelf/
author: Nate Maxwell
categories:
    - "Maya"
tags:
    - "maya"
    - "python"
    - "ui"
    - "shelf"
---


Recently I've been looking into maya shelves. I haven't settled on a good shelf
facility manager for multiple teams to host their own layouts, but I am very
satisfied with this basic shelf class that teams can use to generate their
shelves.

I don't really have much to say on the subject, but I thought I would share
this class here in case anyone is interested.


```python
import os
from pathlib import Path
from typing import Callable

from maya import cmds


def null(*args) -> None:
    pass


# Obviously, change per your project needs
ICONS_PATH = Path('C:/some/icons/path')
DEFAULT_ICON = 'ICON_Default_Blue_40x40.png'


class MayaShelf:
    """
    A simple class to build shelves in maya. Since the build method is empty,
    it should be extended by the derived class to build the necessary shelf
    elements.
    Defaults to creating an empty shelf called 'customShelf'.
    """

    def __init__(self,
                 name: str = 'customShelf',
                 icon_path: Path = ICONS_PATH) -> None:
        """
        Args:
        name(str) -> None: The displayed shelf name in maya. Defaults to
         'customShelf'
        icon_path(str) -> None: The folder path for the shelf's icons.
         Defaults to C:/some/icons/path.
        """
        self.name = name
        self.icon_path = icon_path.as_posix()

        self.labelBackground = (0, 0, 0, 0.5)
        self.labelColor = (.9, .9, .9)

        self._clean_old_shelf()
        cmds.setParent(self.name)
        self.build()
        self._last_item_alignment()

    def build(self) -> None:
        """
        This method should be overwritten in derived classes to actually build
        the shelf elements. Otherwise, nothing is added to the shelf.
        """
        pass

    def add_button(self,
                   label: str,
                   icon: str = DEFAULT_ICON,
                   command: Callable = null,
                   double_command: Callable = null) -> None:
        """Adds a shelf button with the specified label, command, double click
        command, and image.
        """
        cmds.setParent(self.name)
        image = Path(self.icon_path, icon).as_posix()
        if not os.path.exists(image):
            image = 'commandButton.png'

        cmds.shelfButton(width=40, height=40, image=image, l=label,
                         command=command, dcc=double_command,
                         imageOverlayLabel=label, olb=self.labelBackground,
                         olc=self.labelColor, fn='tinyBoldLabelFont')

    def add_menu_item(self,
                      parent: str,
                      label: str,
                      icon: str = '',
                      command=null) -> str:
        """Adds a menu item with the specified label, command, and image."""
        image = Path(self.icon_path, icon).as_posix()
        return cmds.menuItem(p=parent, l=label, c=command, i=image)

    def add_sub_menu(self,
                     parent: str,
                     label: str,
                     icon: str = '') -> None:
        """Adds a sub menu item with the specified label, and optional image,
        to the specified parent popup menu.
        """
        image = Path(self.icon_path, icon).as_posix()
        return cmds.menuItem(p=parent, l=label, i=image, subMenu=1)

    @staticmethod
    def add_separator(style: str = 'none',
                      height: int = 40,
                      width: int = 16) -> str:
        return cmds.separator(st=style, h=height, w=width)

    def _clean_old_shelf(self) -> None:
        """Checks if the shelf exists and empties it if it does, creates it if
        it does not.
        """
        if cmds.shelfLayout(self.name, ex=1):
            if cmds.shelfLayout(self.name, q=1, ca=1):
                for i in cmds.shelfLayout(self.name, q=1, ca=1):
                    cmds.deleteUI(i)
        else:
            cmds.shelfLayout(self.name, p="ShelfLayout")

    def _last_item_alignment(self) -> None:
        """
        Last item is misaligned vertically for some reason, so we
        add a dummy button and remove it. Yay, front end.
        """
        self.add_button('delete_me')
        items = cmds.shelfLayout(self.name, q=1, ca=1)
        cmds.deleteUI(items[-1])
```

This contains an overridable `build()` method where the actual layout and func
connection logic should go.

The `_clean_old_shelf()` method ensures that the previous session's buttons are
removed before rebuilding in case the shelf has been updated between, so that
maya will not retain any old dead and unused buttons.

Example usage

```python
class CustomShelf(MayaShelf):
    def build(self) -> None:
        self.add_button(label="button1")
        self.add_button("button2")
        self.add_button("popup")
        p = cmds.popupMenu(b=1)
        self.add_menu_item(p, "popupMenuItem1")
        self.add_menu_item(p, "popupMenuItem2")
        sub = self.add_sub_menu(p, "subMenuLevel1")
        self.add_menu_item(sub, "subMenuLevel1Item1")
        sub2 = self.add_sub_menu(sub, "subMenuLevel2")
        self.add_menu_item(sub2, "subMenuLevel2Item1")
        self.add_menu_item(sub2, "subMenuLevel2Item2")
        self.add_menu_item(sub, "subMenuLevel1Item2")
        self.add_menu_item(p, "popupMenuItem3")
        self.add_button("button3")
```

Generally, I wrap all my various shelf instantiations in a single function...

```python
def regenerate_shelves() -> None:
    ModellerShelf()
    AnimatorShelf()
    LayoutShelf()
    ProceduralShelf()
    LighterShelf()
    RiggerShelf()
    # etc...
```

...and then call this somewhere in `userSetup.py`.
