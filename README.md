# STEAMING (with LÖVE) ver 1.0.0

### What is this?

*STEAMING (with LÖVE)* is a **s**imple **te**mpl**a**te for **m**ak**in**g **g**ames with *LÖVE2D*. This README is a quick guide to understand some components of this template, and how to use it.

Downloading  the clean template, you already have folders, tables and a whole enviroment ready, improving those slow first steps when creating a game. *STEAMING* most useful aspect is the drawing method (further explained below) that is already organized, filled with useful methods, to facilitate your life managing objects on or offscreen, and an easy way to create and handle objects in your game.

Downloading the example template, you can see a simple "game" to show all those elements in action, and the recommended way to use *STEAMING*'s full potential.

*STEAMING* uses [*HUMP*](http://hump.readthedocs.io/en/latest/), a really awesome library for *LÖVE* that already has classes, timers, gamestates, camera and several other useful stuff implemented. Really recommend reading their documentation a bit to better understand *STEAMING* (and to improve your life!).

### TOO LONG, DON'T WANT TO READ

If you don't want to read this readme, just go ahead and dive in the clean template or the example template. Both are well documented (or so I hope...), and are made to be easily understood. You can check this *README* or other *READMEs* in the template for further explanations anytime you need (don't forget the [*HUMP* documentation](http://hump.readthedocs.io/en/latest/) as well). For a little more indepth of the template, just continue reading.

### Drawing Stuff

``` lua
DRAW_TABLE = {
L1 = {}, --Layer 1 (bottom layer, first to draw)
L2 = {}, --Layer 2
L3 = {}, --Layer 3
L4 = {}, --Layer 4
L5 = {}, --Layer 5
L6 = {}  --Layer 6 (top layer, last to draw)
}

--Draws every drawable object from all tables
function draw.allTables()

    DrawTable(DRAW_TABLE.L1)

    DrawTable(DRAW_TABLE.L2)

    CAM:attach() --Start tracking camera

    DrawTable(DRAW_TABLE.L3)

    DrawTable(DRAW_TABLE.L4)

    DrawTable(DRAW_TABLE.L5)

    CAM:detach() --Stop tracking camera

    DrawTable(DRAW_TABLE.L6)

end

--Draw all the elements in a table
function DrawTable(t)

    for o in pairs(t) do
        if not o.invisible then
          love.graphics.setShader(o.shader) --Set object shader, if any
          o:draw() --Call the object respective draw function
          love.graphics.setShader() --Remove shader, if any
        end
    end

end
```

All the drawing made in your project will be made by Drawing Tables. Those are several tables you can add to the global **DRAW_TABLE** table. In every draw loop, your current gamestate should iterate through all **DRAW_TABLE** tables (done by the method **allTables()** in the *draw* module), and draw all elements inside those tables that are not "invisible" (see **Objects** section below). This way you can easily distribute your visual elements in layers and have them being draw in the order you desire. Another useful function is **DRAW_TABLE(t)**, which draws all elements in a single table t.

If you are using HUMP's awesome camera functions, you can attach or detach the camera during the steps of the **allTable()** method to apply translation effects only in certain layers, but keeping others, such as your *GUI* layer untouched.

You can change the name of the tables to be more explicit, such as renaming your last layer to *GUI* and your first to *BG*. Also, by default there are 6 layers, but you can easily change this to whatever is best for you.

But how do we draw each different element in the Drawing Tables? That's where *HUMP*'s classes come in!

### Objects

```lua
--Element: has a type, subtype and id
ELEMENT = Class{
    init = function(self, _tp, _subtp, _id)
        tp = _tp          --Type this element belongs to
        subtp = _subtp    --Subtype this element belongs to, if any
        id = _id          --Id of this element, if any
        exception = false --If this object is not to be removed when clearing tables
        invisible = false --If this object is not to be draw
    end,

    --other methods here--
}

--Positionable: has a x and y position
POS = Class{
    init = function(self, _x, _y) --Set position for object
        self.x = _x or 0 --X position
        self.y = _y or 0 --Y position
    end,

    setPos = function(self, _x, _y) --Set position for object
        self.x, self.y = _x, _y
    end
}

--Rectangle: is a positionable and colorful object with width and height
RECT = Class{
    __includes = {ELEMENT, POS, CLR},
    init = function(self, _x, _y, _w, _h, _c) --Set rectangle's atributes
        ELEMENT.init(self)
        POS.init(self, _x, _y)
        self.w = _w or 10
        self.h = _h or 10
        CLR.init(self, _c)
    end,

    resize = function(self, _w, _h) --Change width/height
        self.w = _w
        self.h = _h
    end

}

```

*STEAMING* is very friendly to object oriented programming. All your game objects should be instantiated from a class, and those can be organized in the *classes* folder. To draw an object, you specify as a class method his *draw()* method, so that all objects are drawn the way you want them to.

To help you, there is already some primitive classes you can inherit from in the *primitive* module, such as **POS** for an object with and (x,y) position or **CLR** if your object has a main color to be used when it is being drawn. Some premade classes such as **RECT** or **CIRC** are already made, which are for creating simple rectangle and circle objects. Both of them inherit from *ELEMENT*, one of the most important parts of *STEAMING*, that add a *type*, and if you desire, a *subtype* and *id* for any object you may create. This way you can easily find, modify or delete any object, whatever table he may be. Also, elements have an *exception* atribute (see **Gamestates** section), and an *invisible* atribute, that refrains one object from being drawn (*util* module has some useful methods to apply invisible exception to objects).

When applying a subtype for an object (*o:setSubTp("enemy")* for example), if there isn't one, a unique table is created (*SUBTP_TABLE["enemy"]* in the same example), that you can easily iterate or manipulate. Some useful method are already implemented in the *util* module, but you can easily get all objects from a subtype with *SUBTP_TABLE["<subtype name here"]*. Similarly, you can apply an id for an object *o* with *o:setId("wall#48")*, that it will be added to a ID table for quick reference. Again, you can use the *util* module functions to get an specific object with a certain Id, or you can just get it with *ID_TABLE["id name here"]*.

Lastly, you can easily delete an object *o* from all tables with *o:destroy()* or just use the simple methods on the *util* module.

*As a reminder, when changing an object from one table, it will be changed in other tables, since in all these cases you are getting a reference to the object itself.*

For doubts on how to use classes, read the [documentation](http://hump.readthedocs.io/en/latest/class.html).

### Gamestates

```lua
--Called when entering this gamestate
function state:enter()
    local b --button to be created
    local p -- player to be created

    --      x   y  width height  color        function     text     font
    b = But(10, 10, 200, 300, Rgb.orange(), exitfunction, "QUIT", my_font)
    --Add the button to the "GUI" Draw Table, with subtype "gui" and id "ext"
    b:addElement(DRAW_TABLE.GUI, "gui", "ext")
    --          x   y   hp
    p = Player(20, 20, 100)
    --Add the player to the L3 Draw Table, with id "main_character"
    p:addElement(DRAW_TABLE.L3, nil , "main_character")

end

--Called when leaving this gamestate
function state:leave()

    Util.clearAllTables() --Remove all elements without exception

end

--Called every frame, use for logical stuff
function state:update(dt)

    Player.update()

end

--Called every framed, used to draw stuff
function state:draw()

    Draw.allTables()

end
```

HUMP already has everything you'd (probably) need for gamestates, such as changing from one another, and calling the respective callbacks functions. Read more in the [documentation](http://hump.readthedocs.io/en/latest/gamestate.html).

When changing between gamestates, you'll probably want to delete all the draw elements, or at least most of them. To do that you can just call the method *clearAllTables()* in the *util* module, that will clear all elements, including from id's or subtype tables. But if you don't want some objects to be deleted, just use the exceptions methods from the *util* module, to add exceptions to single elements, or even a whole subtype of them. This help you to re-use objects and stop wasting time re-creating them each time you jump from gamestates.

### Useful stuff

There are already some useful classes implemented, such as *rgb* for color manipulation and buttons (by the fault its a rectangle with some text). Feel free to edit and change any files as you desire to better suit your project :)

### How to use

You can fork the repository and use the way you like. If there is a big update on the template, you can re-base your repository, but be careful not to lose specific changes you've made.
