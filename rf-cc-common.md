Chapter 28. Common Cell Commands
=================================

pin
-----

pin - Adds a comment to the pinboard.   

synopsis
---------

pin <comment>

Arguments

comment

    A string which is added to the pinboard. 

Description
-----------

info
-----

info -Print info about the cell.

info [-a] [-l]

Arguments

-a

    Display more information. 
-l

    Display long information. 

Description
-----------

The info printed by `info` depends on the cell class.

dump pinboard
--------------

dump pinboard - Dump the full pinboard of the cell to a file.

synopsis
---------
dump pinboard <filename>

Arguments

filename

    The file the current content of the pinboard is stored in. 
The file the current content of the pinboard is stored in.

Description
-----------

show pinboard
--------------

show pinboard - Print a part of the pinboard of the cell to STDOUT.

show pinboard [ <lines> ]

Arguments

lines

    The number of lines which are displayed. Default: all.

Description
-----------
