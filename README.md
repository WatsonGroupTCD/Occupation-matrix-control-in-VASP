# Occupation-matrix-control-in-VASP

Occupation matrix control sets the occupations used when calculating the DFT+U corrections. It does not directly change the electron distribution. In this way it will effectively encourage the occupations entered. This has two direct uses

1) To set a specific electronic occupation.

2) To obtain localisation at a specific site (atom and location) and look for the minimum energy occupation.

More details can be found in the pdf in each directory.

26/06/2019  Version 1.2 - added support for vasp.5.4.4 

05/09/2019  Version 1.4 - includes all previous functions with the the addition of of spin orbit.
