---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Usage Guide
nav_order: 2
permalink: /usage-guide/
---


# Table of Contents
{:.no_toc}

1. TOC
{:toc }

<br>


While this guide is mostly geared towards programmers new to Lua and modding in general, I highly recommend you read [Organization](#organization) for best practices in mod organization.

If you find any mistakes, please let me know.

# Menu Creation

## Preparing main.lua

To begin, we will need to add a few things to the `main.lua` file.

At the very top, we need to include `kmc_gui.lua` and `helper.lua`. `helper.lua` is a small set a functions that assist with debugging and number manipulation that the GUI uses. If you have these files in folder that isn't called `lib`, you will need to change it accordingly.

*Important note: if you misspell a file or point it to the wrong path, the mod will silently fail without you knowing. Keep this in mind to prevent future frustration.*

```lua
#include "lib/kmc_gui.lua"
#include "lib/helper.lua"
```

We will then need to add `GUI.tick()` to the `draw()` function that Teardown provides.

```lua
function draw()
    GUI.tick()
end
```

Finally, we will need to set the keys that the menu is going to use. This can be done inside the `init()` function that Teardown provides. I recommend creating a function called `initGUI()` and storing all of the variables you use in a table.

```lua
g = {}

function init()
	g.initGUI()
end

g.initGUI = function()
    GUI.navButtons = {
        menuOpen = {
            keys = {"m"},
            menu = "main"
        },
        menuClose = {keys = {}},
        navUp = {keys = {"i", "up"}},
        navDown = {keys = {"k", "down"}},
        select = {keys = {"l", "return", "right"}},
        navBack = {keys = {"j", "left"}}
    }
    
    
    --menu creation stuff goes here
end

```

## Creating a Menu With Buttons

Below is how you would create a menu with buttons. 

```lua
g = {}
g.menuX = 10
g.menuY = 10
g.buttonWidth = 225
g.buttonHeight = 50
g.buttonSpacing = 3
g.fontSize = 23
g.fontName = "bold.ttf"
g.saveLastButtonPos = true

GUI.createMenu("main", nil, "Mod Menu", g.menuX, g.menuY, g.buttonWidth, g.buttonHeight, g.buttonSpacing, g.fontSize, g.fontName, g.saveLastButtonPos)
GUI.addButton("main", "Player Menu >", GUI.setActiveMenu, "player_functions", false, g.fontSize, g.fontName)
GUI.addButton("main", "Sub Menu 2 >", GUI.setActiveMenu, "submenu2", false, g.fontSize, g.fontName)
GUI.addButton("main", "Empty Menu >", GUI.setActiveMenu, "emptymenu", false, g.fontSize, g.fontName)
GUI.addButton("main", "Exit", GUI.setActiveMenu, nil, false, g.fontSize, g.fontName)
```


When calling `GUI.createMenu`, you must specify `menuName`, `prevMenu`, `title`, `menuX`, `menuY`, `buttonWidth`, `buttonHeight`, `buttonSpacing`, `fontSize`, `fontName`, `saveLastButtonPos`. All of these variables are explained under [GUI.createMenu(…)](api-reference.html#guicreatemenu).

When calling `GUI.addButton`, you must specify `parentMenu`, `text`, `funct`, `args`, `toggleable`, `textScale`, `font`. All of these variables are explained under [GUI.addButton(…)](api-reference.html#guiaddbutton).

It is helpful to store these variables in a table (with the exception of `menuName`, `prevMenu`). It lets you recall them later in your mod, and you don't have to name them funny variables to keep track.

Continue reading to [Organization](#organization) for how everything plays together.

## Linking a Button to Another Menu (creating a sub-menu)

It's very helpful to have sub-menus to help break up different function groups. When you call `GUI.addButton`, you can feed it a callback function (the `funct` argument) - this is the function that gets called when the user presses select on it. 

It is recommended for a sub menu, that you add " >" to the end of whatever the button text says. This way it is clear it is a sub-menu, and not a function.

The second argument in `GUI.createMenu` tells the API what menu it should navigate to when the user presses the `navBack` key when the user is in that menu. Notice below where we create `"player_functions"`, the second argument is `"main"`. There is no limit to the amount of sub-menus that you create, the only important part is that they point to each other in an order that makes logical sense (e.g. the user tries going back, and navigates to a totally unrelated vehicle menu).

Setting the second argument to `nil` or `""` will close the menu when the user presses `navBack`.

```lua
GUI.createMenu("main", nil, "Mod Menu", g.menuX, g.menuY, g.buttonWidth, g.buttonHeight, g.buttonSpacing, g.fontSize, g.fontName, g.saveLastButtonPos)
GUI.addButton("main", "Player Menu >", GUI.setActiveMenu, "player_functions", false, g.fontSize, g.fontName)

GUI.createMenu("player_functions", "main", "Player Menu", g.menuX, g.menuY, g.buttonWidth, g.buttonHeight, g.buttonSpacing, g.fontSize, g.fontName, g.saveLastButtonPos)
GUI.addButton("player_functions", "Exit", GUI.setActiveMenu, nil, false, g.fontSize, g.fontName)
```

# Best Practices

## Organization
Once you've created a group of menus and each button has a dedicated function - things can become complicated rapidly without an organization standard. When you declare a global variable inside Lua, *any other included .lua file can access that variable*. As you can imagine, sometimes you have to create a variable with the same name. Naturally, this will create problems of variables writing over each other (thus creating hard-to-diagnose bugs, especially considering error handling in-game doesn't give a full stack trace at the time of writing). This is where the idea of separating everything into tables comes into play. This isn't quite object-oriented programming - more like a distant cousin.

### File Layout

Let's say we want to have 3 groups of functions in our mod: `player_functions`, `vehicle_functions`, `world_functions`. Inside the `/My Mod Menu/functions` folder, we create a `.lua` file for each of these functions (see [Recommended Folder Structure](#recommended-folder-structure)).


```
.
└── Mod_Menu_Folder/
    └── functions/
        ├── player_functions.lua
        ├── vehicle_functions.lua
        └── world_functions.lua

```


### Variable Layout

At the top of `player_functions.lua` we create a global table, and throughout the rest of the file, we add to that table. I recommend below as a base file:

#### Base_Function_File.lua

```lua
--player_functions.lua

player_functions = {}

player_functions.tick = function()
	--Do stuff every tick
end

player_functions.init = function()
	--Do stuff once when the mod has started
end
```

Inside [main.lua](https://www.teardowngame.com/modding/#main.lua) we can include `player_functions.lua`, then call `player_functions.tick` and `player_functions.init`. Note: for simplicity,  I haven't included setting up the menu.

```lua
--main.lua

#include "lib/kmc_gui.lua"
#include "lib/helper.lua"

#include "functions/player_functions.lua"

g = {}

function init()
	g.initGUI() --see Menu Creation for what this should look like

	player_functions.init()
end

function tick()
	player_functions.tick()
end

function draw()
    GUI.tick()
end
```

After this, we can start adding functions and variables:

#### Example_function_file.lua
```lua
--player_functions.lua

player_functions = {}

player_functions.togglesomething = function()
    if(GUI.isActiveButtonToggledOn()) then
        player_functions.toggled = true
        GUI.updateStatusText("Enabled some really cool function!" 2.5)
    else
        player_functions.toggled = false
        GUI.updateStatusText("Disabled some really cool function!" 2.5)
        DebugPrint("Toggle turned off!")
    end
end

player_functions.tick = function()
    if(player_functions.toggled) then
    	--Do something every tick
        DebugPrint("Toggle turned on!")
    end
end

player_functions.init = function()
	player_functions.toggled = false

	GUI.createMenu("player_functions", "main", "Player Functions", g.menuX, g.menuY, g.buttonWidth, g.buttonHeight, g.buttonSpacing, g.fontSize, g.fontName, g.saveLastButtonPos)
	GUI.addButton("player_functions", "Toggle", player_functions.togglesomething, nil, true, g.fontSize, g.fontName)
end
```


## Recommended Folder Structure

It is recommended you follow this structure below. Any functions you add should go into the `functions/` folder. See (see [File Layout](#file-layout) for more details).

```
.
└── Mod_Menu_Folder/
    ├── lib/
    │   ├── helper.lua
    │   └── kmc_gui.lua
    ├── functions/
    │   ├── example1_functions.lua
    │   ├── example2_functions.lua
    │   └── example3_functions.lua
    ├── media/
    │   ├── tick_down.ogg
    │   └── tick_up.ogg
    ├── main.lua
    ├── info.txt
    └── options.lua (optional)
```