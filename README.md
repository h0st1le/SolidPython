SolidPython
-----------
- [SolidPython:  OpenSCAD for Python](#solidpython--openscad-for-python)
- [Advantages](#advantages)
- [Installing SolidPython](#installing-solidpython)
- [Using SolidPython](#using-solidpython)
- [Example Code](#example-code)
- [Extra syntactic sugar](#extra-syntactic-sugar)
	- [Basic operators](#basic-operators)
- [solid.utils](#solidutils)
	- [Directions: (up, down, left, right, forward, back) for arranging things:](#directions-up-down-left-right-forward-back-for-arranging-things)
	- [Arcs](#arcs)
	- [Basic color library](#basic-color-library)
	- [Bill Of Materials](#bill-of-materials)
- [Contact](#contact)

**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*


SolidPython:  OpenSCAD for Python<a id="solidpython--openscad-for-python"></a>
-----------------------

SolidPython is a generalization of Phillip Tiefenbacher's openscad module, 
found on [Thingiverse](http://www.thingiverse.com/thing:1481).  It generates valid OpenSCAD 
code from Python code with minimal overhead.  Here's a simple example:
    
This Python code: 

    from solid import *
    d = difference()(
        cube(10),
        sphere(15)
    )
    print scad_render( d)
 

Generates this OpenSCAD code:

    difference(){
        cube(10);
        sphere(15);
    }

That doesn't seem like such a savings, but the following SolidPython code is a 
lot shorter (and I think a lot clearer) than the SCAD code it compiles to:

    d = cube( 5) + right(5)( sphere(5)) - cylinder( r=2, h=6)

Generates this OpenSCAD code:

    difference(){
        union(){
            cube(5);
            translate( [5, 0,0]){
                sphere( 5);
            }
        }
        cylinder( r=2, h=6);
    }

Advantages<a id="advantages"></a>
----------
Because you're using Python, a lot of things are easy that would be hard or 
impossible in pure OpenSCAD.  Among these are:

* built-in dictionary types
* mutable, slice-able list and string types
* recursion
* external libraries (images! 3D geometry!  web-scraping! ...)

Installing SolidPython<a id="installing-solidpython"></a>
----------------------
*   Install via [PyPI](python setup.py sdist bdist_wininst upload):

        sudo easy_install solidpython
        

    At time of writing, `pip install solidpython` will NOT work (13 Feb 2013)

*   **OR:** Download SolidPython ( Click [here](https://github.com/SolidCode/SolidPython/archive/master.zip) to download directly, or use git to pull it all down)

    ( Note that SolidPython also depends on the [PyEuclid](http://pypi.python.org/pypi/euclid) Vector math library, installable via `sudo pip install euclid`)
    
    *   Unzip the file, probably in ~/Downloads/SolidPython-master
    *   In a terminal, cd to location of file:
  
            cd ~/Downloads/SolidPython-master

    *   Run the install script: 

            sudo python setup.py --install

Using SolidPython<a id="using-solidpython"></a>
-------------------------
*   Include SolidPython at the top of your Python file:

        from solid import *
        from solid.utils import *  # Not required, but the utils module is useful
*   To include other scad code, call ```use("/path/to/scadfile.scad")``` or ```include("/path/to/scadfile.scad")```
*   OpenSCAD uses curly-brace blocks ({}) to create its tree.  SolidPython uses
    parentheses with comma-delimited lists.
    __OpenSCAD:__
    
        difference(){
            cube(10);
            sphere(15);
        }

    __SolidPython:__
    
        d = difference()(
            cube(10),  # Note the comma between each element!
            sphere(15)
        )
           
*   Call ```scad_render( py_scad_obj)``` to generate SCAD code. This returns a string 
    of valid OpenSCAD code.
*   *or*: call ```scad_render_to_file( py_scad_obj, filepath)``` to
    store that code in a file. 
*   If 'filepath' is open in the OpenSCAD IDE and Design =>
    'Automatic Reload and Compile' is checked (in the OpenSCAD IDE), calling
    ```scad_render_to_file()``` from Python will load the object in
    the IDE.
*   Alternately, you could call OpenSCAD's command line and render straight 
    to STL.   
    
Example Code<a id="example-code"></a>
------------
The best way to learn how SolidPython works is to look at the included example code. 
If you've installed SolidPython, the  following line of Python will print the location of 
the examples directory:

        import os, solid; print os.path.dirname( solid.__file__) + '/examples'
        
Or browse the example code on Github [here](https://github.com/SolidCode/SolidPython/tree/master/examples)

Adding your own code to the example file `solidpython_template.py` will make some of the setup easier.

Extra syntactic sugar<a id="extra-syntactic-sugar"></a>
---------------------
### Basic operators<a id="basic-operators"></a>
Following Elmo Mäntynen's suggestion, SCAD objects override 
the basic operators + (union), - (difference), and * (intersection).
So

    c = cylinder( r=10, h=5) + cylinder( r=2, h=30)
is the same as:

    c = union()(
        cylinder( r=10, h=5),
        cylinder( r=2, h=30)
    )

Likewise:

    c = cylinder( r=10, h=5)
    c -= cylinder( r=2, h=30)

is the same as:

    c = difference()(
        cylinder( r=10, h=5),
        cylinder( r=2, h=30)
    )



solid.utils<a id="solidutils"></a>
-----------
SolidPython includes a number of useful functions in solid/utils.py.  Currently these include:
    
### Directions: (up, down, left, right, forward, back) for arranging things:<a id="directions-up-down-left-right-forward-back-for-arranging-things"></a>
    
    up(10)(
        cylinder()
    )

seems a lot clearer to me than:

    transform( [0,0,10])(
        cylinder()
    )
    
Again, I took this from someone's SCAD work and have lost track of the 
original author.  My apologies.
    
### Arcs<a id="arcs"></a>
I've found this useful for fillets and rounds.

    arc( rad=10, start_degrees=90, end_degrees=210)

draws an arc of radius 10 counterclockwise from 90 to 210 degrees. 

    arc( rad=10, start_degrees=0, end_degrees=90, invert=True ) 

draws the portion of a 10x10 square NOT in a 90 degree circle of radius 10.
This is the shape you need to add to make fillets or remove to make rounds.
    
### Basic color library<a id="basic-color-library"></a>
You can change an object's color by using the OpenSCAD ```color([rgba_array])``` function:

    transparent_blue = color( [0,0,1, 0.5])( cube(10))  # Specify with RGB[A]
    red_obj = color( Red)( cube( 10))                   # Or use predefined colors

These colors are pre-defined in solid.utils:

<table border="0" cellspacing="5" cellpadding="0">
    <tr><td>* Red        </td><td>* Green      </td><td>* Blue       </td></tr>
    <tr><td>* Cyan       </td><td>* Magenta    </td><td>* Yellow     </td></tr>
    <tr><td>* Black      </td><td>* White      </td><td>* Transparent</td></tr>
    <tr><td>* Oak        </td><td>* Pine       </td><td>* Birch      </td></tr>
    <tr><td>* Iron       </td><td>* Steel      </td><td>* Stainless  </td></tr>
    <tr><td>* Aluminum   </td><td>* Brass      </td><td>* BlackPaint </td></tr>
    <tr><td>* FiberBoard </td></tr>
</table>

I took this from someone on Thingiverse and I'm 
ashamed that I can't find the original source.  I owe someone some 
attribution.
    
### Bill Of Materials<a id="bill-of-materials"></a>
Put ```@part()``` before any method that defines a part, then 
call ```bill_of_materials()``` after the program is run, and all parts will be 
counted, priced and reported. 

The example file `bom_scad.py` illustrates this. Check it out.

Contact<a id="contact"></a>
-------
Enjoy, and please send any questions or bug reports to me at ```evan_t_jones@mac.com```. Cheers!
Evan
