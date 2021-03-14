---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: API Reference
nav_order: 3
permalink: /api-reference/

string: "&nbsp;<span class='type string'><a href='https://www.lua.org/pil/2.4.html' target='_blank'>&lt;string&gt;</a></span>&nbsp;"
number: "&nbsp;<span class='type number'><a href='https://www.lua.org/pil/2.3.html' target='_blank'>&lt;number&gt;</a></span>&nbsp;"
bool: "&nbsp;<span class='type bool'><a href='https://www.lua.org/pil/2.2.html' target='_blank'>&lt;boolean&gt;</a></span>&nbsp;"
table: "&nbsp;<span class='type table'><a href='https://www.lua.org/pil/2.5.html' target='_blank'>&lt;table&gt;</a></span>&nbsp;"
function: "&nbsp;<span class='type function'><a href='https://www.lua.org/pil/2.6.html' target='_blank'>&lt;function&gt;</a></span>&nbsp;"

---

Please see [Usage](usage) for more examples and best practices.

# Table of Contents
{:.no_toc}

- TOC
{:toc }

* * *

# Functions

## GUI.createMenu(...)
### `GUI.createMenu(menuName, prevMenu, title, menuX, menuY, buttonWidth, buttonHeight, buttonSpacing, fontSize, fontName, saveLastButtonPos)`
{:.no_toc}

- `menuName` {{ page.string }} Name of the menu that `GUI.createMenu` is creating
- `prevMenu` {{ page.string }} Name of the menu that the `navBack` key should take the user to when inside this menu. Set to `nil` to close the menu altogether.
- `title` {{ page.string }} Text that appears at the top, above the buttons
- `menuX` {{ page.number }} Top left of the menu starting `x` position
- `menuY` {{ page.number }}	Top left of the menu starting `y` position
- `buttonWidth` {{ page.number }} How wide each button should be in the `x` direction
- `buttonHeight` {{ page.number }} How tall each button should be in the `y` direction
- `buttonSpacing` {{ page.number }} How tall the gap should be between each button
- `fontSize` {{ page.number }} How tall the font should be in pixels
- `fontName` {{ page.string }} Name of `.ttf` file to use for the font (suggested `bold.ttf`)
- `saveLastButtonPos` {{ page.bool }} After exiting and re-entering the menu, should the button the user on be remembered? If false, it will be reset back to the first button.
- Returns: nothing

```lua
--the g(lobal) table is used to keep non-tabled global variables from being leaked across multiple lua files. See Usage.
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
```

* * *

## GUI.addButton(...)
### `GUI.addButton(parentMenu, text, funct, args, toggleable, textScale, font)`
{:.no_toc}

- `parentMenu` {{ page.string }} What menu this button belongs to.
- `text` {{ page.string }} Text drawn on the button
- `funct` {{ page.function }} Callback function for when the user presses the `select` key
- `args` {{ page.table }} Up to 20 arguments to feed into the defined `funct`. `{arg1, ..., arg20}`. Can also be a single {{ page.string }}, {{ page.bool }}, or {{ page.number }}
- `toggleable` {{ page.bool }} Should button be a toggleable button? Will have `[ON]` or `[OFF]` appended to the end of the button text, and the state can be grabbed at any time
- `textScale`{{ page.number }} how tall the font should be in pixels (aka font size)
- `font`{{ page.string }} name of `.ttf` file to use for the font (suggested `bold.ttf`)
- Returns: nothing

Adds a button to a menu that has already been created. `parentMenu` must match the name of a menu already created, *otherwise this will not work.*

The position in the menu, and the index of the button are dependent on when the button is added. If you want button `x` to appear before button `y`, you will need to addButton() `x` first.

See [GUI.createMenu](#guicreatemenu) for what `g` can be in the example below.

```lua
GUI.createMenu("main", nil, "Mod Menu", g.menuX, g.menuY, g.buttonWidth, g.buttonHeight, g.buttonSpacing, g.fontSize, g.fontName, g.saveLastButtonPos)
GUI.addButton("main", "Player Menu >", GUI.setActiveMenu, "player_functions", false, g.fontSize, g.fontName)
GUI.addButton("main", "Vehicle Menu >", GUI.setActiveMenu, "vehicle_functions", false, g.fontSize, g.fontName)
GUI.addButton("main", "Empty Menu >", GUI.setActiveMenu, "empty_menu", false, g.fontSize, g.fontName)
GUI.addButton("main", "Exit", GUI.setActiveMenu, nil, false, g.fontSize, g.fontName)
```

* * *

## GUI.setActiveMenu(...)
### `GUI.setActiveMenu(menuName)`
{:.no_toc}
- `menuName` {{ page.string }} Name of the menu the menu that should be drawn currently
- Returns: nothing

Will start drawing `menuName` immediately. Often used as a callback for `GUI.addButton` to create a sub menu. Set `funct = GUI.setActiveMenu` and `args = "NameOfMenu"`. Can also be called at any time.

```lua
GUI.setActiveMenu("main")
--or
GUI.addButton("main", "Player Menu >", GUI.setActiveMenu, "player_functions", false, g.fontSize, g.fontName)
```

* * *

## GUI.tick()
### `GUI.tick()`
{:.no_toc}
- Returns: nothing

*Must be called by draw() in order for the GUI to function.* If you add it to the `tick()` function, it will be called out of order when rendering the menu and won't appear.

```lua
function draw()
    GUI.tick()
end
```

* * *

## GUI.updateStatusText(...)
### `GUI.updateStatusText(text, time)`
{:.no_toc}
- `text`{{ page.string }} Text to display as a message
- `time`{{ page.number }} Time in seconds for how long message should last
- Returns: nothing

Think of this as a toast message. `text` will appear on the screen for `time` in big letters, so the user knows what is happening at a glance.

```lua
GUI.updateStatusText("Enabled God mode!", 2.5)
```

* * *

## GUI.isActiveButtonToggledOn()
### `GUI.isActiveButtonToggledOn()`
{:.no_toc}
- Returns: {{ page.bool }} True if button is set to `[ON]`, false otherwise.

Returns the state of the button the user currently has highlighted. This seems useless at first glance, but it's very handy when a toggle-able button is mapped to a function, and you need to know if you should enable or disable said function.

```lua
main.toggleGodMode = function()
    if(GUI.isActiveButtonToggledOn()) then
        --do something
        GUI.updateStatusText("Enabled God mode", 2.5)
    else
        --do something
        GUI.updateStatusText("Disabled God mode", 2.5)
    end
end

GUI.createMenu("main", "", "Mod Menu", g.menuX, g.menuY, g.buttonWidth, g.buttonHeight, g.buttonSpacing, g.fontSize, g.fontName, g.saveLastButtonPos)
GUI.addButton("main", "God mode", main.toggleGodMode, nil, true, g.fontSize, g.fontName)
```

* * *

## GUI.isButtonToggledOn(...)
### `GUI.isButtonToggledOn(menu, index)`
{:.no_toc}
- `menu`{{ page.string }} Menu that the button belongs to
- `index`{{ page.number }} index of button in `menu`. Index starts at 1 (not 0 like most other languages)
- Returns: {{ page.bool }} True if button is set to `[ON]`, false otherwise.

Similar to `GUI.isActiveButtonToggledOn()`, except you can check a specific button instead.

```lua
local toggled = GUI.isButtonToggledOn("player_functions", 3)
```

* * *

## GUI.toggleButtonState(...)
### `GUI.toggleButtonState(menu, index)`
{:.no_toc}
- `menu`{{ page.string }} Menu that the button belongs to
- `index`{{ page.number }} index of button in `menu`. Index starts at 1 (not 0 like most other languages)
- Returns: nothing

Gives you the ability to manually toggle a button on or off. Useful if other buttons rely on a different button to be on or off.

```lua
some_cool_menu.turnOffAbilities = function()
	if(GUI.isButtonToggledOn("some_cool_menu", 5)) then
		GUI.toggleButtonState("some_cool_menu", 5)
	end

	if(GUI.isButtonToggledOn("some_cool_menu", 6)) then
		GUI.toggleButtonState("some_cool_menu", 6)
	end
end

some_cool_menu.turnOffAbilities()
```

* * *

## GUI.removeButton(...)
### `GUI.removeButton(menu, index)`
{:.no_toc}
- `menu`{{ page.string }} Menu that the button belongs to
- `index`{{ page.number }} index of button in `menu`. Index starts at 1 (not 0 like most other languages)
- Returns: nothing

Removes button `index` from `menu`. Useful for menus that are actively changing (e.g. a list of players currently in the session) 

```lua
GUI.removeButton("some_cool_menu", 4)
```

* * *

## GUI.removeMenu(...)
### `GUI.removeMenu(menu)`
{:.no_toc}
- `menu`{{ page.string }} Menu that is being deleted
- Returns: nothing

Sets `menu` to `nil` - making it unable to be drawn again.

```lua
GUI.removeMenu("some_uncool_menu")
```

***

## _These functions have been documented for reference, but should not be called. They are internal functions for the GUI library._
{:.no_toc}

<div class="code-example" markdown="1">

## GUI.toggleActiveButtonState()
{:.no_toc}
### `GUI.toggleActiveButtonState()`
{:.no_toc}
- Returns: nothing

You should not have to call this function. It is used internally to toggle a button when the user presses `select`.

* * *

## GUI.drawStatusText()
{:.no_toc}
### `GUI.drawStatusText()`
{:.no_toc}
- Returns: nothing

Do not call this function. It is used internally to draw the status text on the screen.

* * *

## GUI.drawGUI()
{:.no_toc}
### `GUI.drawGUI()`
{:.no_toc}
- Returns: nothing

Do not call this function. It is used internally to draw the buttons on screen.

* * *

## addButtonText(...)
{:.no_toc}
### `GUI.addButtonText(text, xpos, ypos, textScale, font, [color], [xOverride], [yOverride], [toggleable], [statusText])`
{:.no_toc}
- Returns: {{ page.table }} {textW, textH, statusW} - used to center the text on the buttons

Do not call this function. It is used internally to draw text on the buttons programmatically.

* * *

## GUI.runButtonFunct(...)
{:.no_toc}
### `GUI.runButtonFunct(currButton)`
{:.no_toc}
- `currButton` {{ page.table }} Takes a `button` table as an input
- Returns: {{ page.bool }} True if called the function successfully

Do not call this function. It is used internally to call the functions set for each button.


</div>
* * *

# Variables

## GUI.navButtons = { ... }

`GUI.navButtons` is designed to take 0, 1, or multiple keys as variables. The `keys` variable is a table of {{ page.string }}. A list of available keys can be found on the [Teardown API](https://www.teardowngame.com/modding/api.html).

`GUI.menuOpen.menu` is the menu {{ page.string }} that will open when any of the `menuOpen` keys are pressed. 

### Definition:
{:.no_toc}
```lua
GUI.navButtons = {
	menuOpen = {
		keys = {<string>},
		menu = <string>
	}, 
	menuClose = {keys = {<string>}},
	navUp = {keys = {<string>}},
	navDown = {keys = {<string>}},
	select = {keys = {<string>}},
	navBack = {keys = {<string>}}
}
```

### Example:
{:.no_toc}
```lua
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
```

* * *

## GUI.titleColor = { ... }

Sets the color for the menu title via RGBA. The color is for every menu created. A tad bit of transparency `a` is recommended so the player can see what's behind the menu. 1 is completely opaque, 0 is transparent. Set this when initiating the GUI.

### Definition:
{:.no_toc}
```lua
GUI.titleColor = {
	r = <number>[0-255],
	g = <number>[0-255],
	b = <number>[0-255],
	a = <number>[0-1]
}
```

### Example:
{:.no_toc}
```lua
GUI.titleColor = {
	r = 255, 
	g = 100, 
	b = 100, 
	a = .9
}
```

* * *

## GUI.buttonColor = { ... }

Sets the color of the buttons via RGBA. The color is across every menu created. A tad bit of transparency `a` is recommended so the player can see what's behind the menu. 1 is completely opaque, 0 is transparent. Set this when initiating the GUI.

### Definition:
{:.no_toc}
```lua
GUI.buttonColor = {
	r = <number>[0-255],
	g = <number>[0-255],
	b = <number>[0-255],
	a = <number>[0-1]
}
```

### Example:
{:.no_toc}
```lua
GUI.buttonColor = {
	r = 20,
	g = 20,
	b = 20,
	a = .9
}
```

* * *

## GUI.activeButtonColor = { ... }

Sets the color of the button currently active (the one the user has selected) via  via RGBA. The color is across every menu created. A tad bit of transparency `a` is recommended so the player can see what's behind the menu. 1 is completely opaque, 0 is transparent. Set this when initiating the GUI.

This color should be different from [GUI.buttonColor](#buttonColor--).

### Definition:
{:.no_toc}
```lua
GUI.activeButtonColor = {
	r = <number>[0-255],
	g = <number>[0-255],
	b = <number>[0-255],
	a = <number>[0-1]
}`
```

### Example:
{:.no_toc}
```lua
GUI.activeButtonColor = {
	r = 20,
	g = 20,
	b = 20,
	a = .9
}
```
* * *

## GUI.sound = { ... }

GUI.sound.tick_down is played when the user navigates up or down.

GUI.sound.tick_up is played when the user presses select.

See the game API on [LoadSound](https://www.teardowngame.com/modding/api.html#LoadSound).

### Example:
{:.no_toc}
```lua
GUI.sound = {
	tick_up = LoadSound("media/tick_up.ogg"),
	tick_down = LoadSound("media/tick_down.ogg")
}
```
