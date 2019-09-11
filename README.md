# Occupation-matrix-control-in-VASP

Occupation matrix control sets the occupations used when calculating the DFT+U corrections. It does not directly change the electron distribution. In this way it will effectively encourage the occupations entered. This has two direct uses

1) To set a specific electronic occupation.

2) To obtain localisation at a specific site (atom and location) and look for the minimum energy occupation.

More details can be found in the pdf in each directory.

26/06/2019  Version 1.2 - added support for vasp.5.4.4 

05/09/2019  Version 1.4 - includes all previous functions with the the addition of of spin orbit.

## Installation

for the latest versino place the patch files `Occupation_matrix_vasp.5.4.4_spin_oribit.patch` in the VASP root directory 

```
$ patch -p0 < Occupation_matrix_vasp.5.4.4_spin_oribit.patch 
```

You can then compile vasp in the usual way. 


Note - For older versions separate patch files are provided for main.F and LDAu.F which must be patched from the src directory - see the instructions in the associated pdf for further details.

## Usage 

Occupation matrix control sets the occupations used when calculating the DFT+U corrections. It does not directly change the electron distribution. In this way it will effectively encourage the occupations entered. This has two direct uses

1) To set a specific occupation.
In this case the electron occupations are set and the structure relaxed. It is usually best in the first step to ensure localisation on the site of choice as the initial localisation will direct the associated polaronic distortion at the site during relaxation.  Once the relaxation is complete (to some degree) the occupation control should be switched off and the calculation restarted. The occupation can be picked up from the WAVECAR and the system is allowed to relax. The system may or may not retain the occupation originally intended and this will need to be checked. LDAUPRINT=2 is useful for this. 
The reason for this is that using the occupation matrix causes shifts in the energy as you are setting occupations - which even for the specific electron configuration you are looking at is never absolutely correct (there is always some mixing / distortion / rotation  of orbitals). Hence the need to restart with the occupation matrix switched off. 

2) To obtain localisation at a specific site and look for the minimum energy occupation.
This is slightly easier as essentially you are attempting to create a polaronic distortion. In simple systems like TiO2 and CeO2 this can be done by setting a single electron occupation (usually use one of the low energy orbitals for the crystal field) and relaxing the system. Again it is usually best in the first step to ensure localisation on the site of choice as the initial localisation will direct the associated polaronic distortion at the site during relaxation. Once it has relaxed and created the distortion (do not usually need it that well converged) you can switch the occupation matrix control off. Generally we have found (especially for f orbitals) that the occupation becomes trapped in a specific orbital while in some d electron systems the electron can sometimes switch orbitals. 

For d orbitals, if you want the lowest energy it is usually best to delete the WAVECAR and allow VASP to search for the lowest electronic structure for the given distortion . It is also advised to perform a single SCF initially and ensure reasonable localisation before relaxing the structure. 

For f orbitals we have sometimes found that the distortions for different orbitals can be sufficiently different that it can direct the occupation even with the WAVECAR deleted. Hence, you have to search (or at least guess) which orbital should be occupied in the initial optimisation (i.e. choose a low energy orbital for the crystal field splitting). 


### Specifiying Occupations 

Occupation matrix control can be used for any orbital - s, p, d or f - although VASP only allows U to be applied to a single set of orbitals (s, p, d or f) for a given atom. Each atom essentially has a matrix of orbital occupations - VASP defines this with respect to the Cartesian coordinates. This can make getting specific occupation (or orbitals) difficult if the local structure of the atom does not correspond to the x, y and z axes. 

There are two different methods for specifiying occupations 

#### OCCDIRX

This allows specific orbital occupations to be specified in the INCAR files using the keyword OCCDIRX - where X is a number between 1 and 999. 

```
OCCDIRX        No of atoms, No * (ATOM, LDAUL, No of elements, No of E *( i, j, spin, occ))
```

No of atoms 	- 	number of atoms specified on this line 
Atom       	- 	atom number in POSCAR
LDAUL       	- 	orbital type (0-s, 1-p, 2-d, 3-f)
No of elements 	-	Number of occupation matrix elements set for this atom 
i,j        	- 	matrix element (1-16,1-16)
spin        	- 	which spin the occupation is for (1-up, 2-down)
occ 		- 	the occupation (up to 1.0) 

NOTE - Anything not set by the OCCDIRX lines will automatically be set to zero. 

E.g.
```
    OCCDIR1 = 2 1 3 1 13 13 1 1.0 2 3 1 13 13 1 1.0
```

Occupations are set on two atoms:
 	Atom 1 in POSCAR - f orbital, 1 element to set: 13-13 (f-3) up spin occupation set to 1.0
 	Atom 2 in POSCAR - f orbital, 1 element to set: 13-13 (f-3) up spin occupation set to 1.0


NOTE - Non-colinnear input does not include the imaginary component of the matrix elements and is therefore not recommended. 

#### OCCEXT     

We have implemented a direct occupation matrix input for atoms through an external file. This greatly helps when 
1) the crystal field is not aligned with the axis of the structure (in which case a number of off diagonal elements are required)
2) dealing with atoms with large numbers of d and f electrons
3) there is a specific orbital distortion (e.g. due to vacancies or at surfaces) 
4) there is significant co-valent interactions resulting in more d occupation than simply the filling of specific orbitals.
To use this you need a good guess of the occupation matrix appropriate for the environment. This can often be achieve by using OCCDIR on a simple unit cell or taken from another crystal structure (e.g. for a particular oxidation state) 

Reading in from an external file (OCCMATRIX) is specified in the INCAR by the tag OCCEXT = 1. 

The format of the file is similar to the output when LDAUPRINT = 2 is specified (see example below). The top line has the number of atoms which have their occupations set in the external file. There are then this number of blocks on input with the following format. The first line of each block has the atom number (in the POSCAR), the angular momentum of the orbitals in question and whether there is spin(2) or not(1). Following this is a line which I usually just use to label the spin component (which is not actually taken notice of). Following this is the spin up occupation matrix (size appropriate for the angular momentum). If spin polarization was requested this is followed by another line for comment and a second occupation matrix. There is a blank line between the end of one block and the start of the next.  

For example the occupations of Fe3+ (d5)  taken from a FM calculation can be used to specific an AFM arrangement with the matrices for the spin up and spin down reversed for the second atom (I left the spin label to remind me a reversed them).

```
2                             No of atoms to be specified 
3   2 2                       Atom No (in POSCAR),  L,  spin(2) or not (1)
spin component  1			Not really important what is here ! 
   0.7997  0.3554 -0.0000 -0.0000 -0.0000
   0.3554  0.5269 -0.0000 -0.0000 -0.0000
  -0.0000 -0.0000  1.0225  0.0653  0.0575
  -0.0000 -0.0000  0.0653  0.7468 -0.3435
  -0.0000 -0.0000  0.0575 -0.3435  0.6179
spin component  2			
   0.9057 -0.3651  0.0000 -0.0000  0.0000
  -0.3651  0.3291  0.0000 -0.0000  0.0000
   0.0000  0.0000  0.5578  0.4221  0.0993
  -0.0000 -0.0000  0.4221  0.5288  0.2056
   0.0000  0.0000  0.0993  0.2056  0.2269
                             Blank line between atoms 
4   2 2                      Atom No (in POSCAR),  L,  spin(2) or not (1)
spin component  2			Spins switched to give the AFM arrangmnet
   0.9057 -0.3651  0.0000 -0.0000  0.0000
  -0.3651  0.3291  0.0000 -0.0000  0.0000
   0.0000  0.0000  0.5578  0.4221  0.0993
  -0.0000 -0.0000  0.4221  0.5288  0.2056
   0.0000  0.0000  0.0993  0.2056  0.2269
spin component  1
   0.7997  0.3554 -0.0000 -0.0000 -0.0000
   0.3554  0.5269 -0.0000 -0.0000 -0.0000
  -0.0000 -0.0000  1.0225  0.0653  0.0575
  -0.0000 -0.0000  0.0653  0.7468 -0.3435
  -0.0000 -0.0000  0.0575 -0.3435  0.6179
					Blank line 
```

#### Non-colinear / spin orbit calculations

in the case of non-collinear / spin orbit the occupation matrix is now twice as large as the number of atomic orbital involved. In addition, the matrix is complex. VASP handles this by having 4 complex n x n matrices, where n is the number of atomic orbital relevant for the given L. e.g. a four 5x5 matrices for d orbitals or four 7x7 matrices for f orbitals. 
The matrix input follows the same format as for spin polarised calculation except instead of two spins the spin tag is used to select the four matrices input. In addition there are two matrices side by side which are the real and complex parts of the occupation matrix. 

The example below shows the occupation matrix for the 3K magnetic structure of UO2. There are four atoms in the cubic unit cell with each with 4 “spin components” which represent the different parts of the single occupation matrix

```
4
1 3 4                                
spin component  1
  0.2942  0.0147  0.0789  0.0078  0.0297 -0.0015 -0.0876     -0.0000  0.0006  0.0021  0.0039  0.0215 -0.2219 -0.0922
  0.0147  0.1290  0.0137  0.0186  0.0035  0.0020 -0.0045     -0.0006  0.0000  0.0188 -0.0116 -0.0147 -0.0165 -0.0121
  0.0789  0.0137  0.3010  0.0060 -0.0652  0.0030 -0.0285     -0.0021 -0.0188 -0.0000 -0.2886 -0.0733 -0.0890 -0.0242
  0.0078  0.0186  0.0060  0.3732  0.0933  0.0226 -0.0062     -0.0039  0.0116  0.2886  0.0000 -0.0864 -0.0013 -0.0048
  0.0297  0.0035 -0.0652  0.0933  0.0832 -0.0134 -0.0212     -0.0215  0.0147  0.0733  0.0864  0.0000 -0.0168 -0.0044
 -0.0015  0.0020  0.0030  0.0226 -0.0134  0.2214  0.0798      0.2219  0.0165  0.0890  0.0013  0.0168 -0.0000 -0.0738
 -0.0876 -0.0045 -0.0285 -0.0062 -0.0212  0.0798  0.0945      0.0922  0.0121  0.0242  0.0048  0.0044  0.0738 -0.0000
spin component  2
  0.1042  0.0116  0.0345  0.0033  0.0022  0.0827 -0.0006      0.0966  0.0055  0.0302 -0.0023  0.0168 -0.0904 -0.0713
  0.0049  0.0041  0.0264  0.0051 -0.0025 -0.0068 -0.0065     -0.0083 -0.0040  0.0020 -0.0210 -0.0073 -0.0166 -0.0005
  0.0015  0.0017  0.0585  0.0809  0.0042  0.0178  0.0030      0.0130  0.0001  0.0717 -0.0663 -0.0359 -0.0047 -0.0048
 -0.0115 -0.0043  0.0799 -0.1147 -0.0596 -0.0018  0.0066      0.0060 -0.0097 -0.0976 -0.0983  0.0025  0.0007  0.0037
 -0.0095  0.0152  0.0134  0.2741  0.0759 -0.0437 -0.0251     -0.0758  0.0038  0.2104 -0.0165 -0.0836  0.0118  0.0259
  0.0673  0.0033 -0.0087  0.0162  0.0278 -0.0635 -0.0477     -0.0757  0.0048 -0.0094  0.0316  0.0030 -0.0544  0.0013
 -0.0186 -0.0002 -0.0053 -0.0007 -0.0237  0.2309  0.1020      0.2714  0.0163  0.0829  0.0056  0.0266  0.0147 -0.0837
spin component  3
  0.1042  0.0049  0.0015 -0.0115 -0.0095  0.0673 -0.0186     -0.0966  0.0083 -0.0130 -0.0060  0.0758  0.0757 -0.2714
  0.0116  0.0041  0.0017 -0.0043  0.0152  0.0033 -0.0002     -0.0055  0.0040 -0.0001  0.0097 -0.0038 -0.0048 -0.0163
  0.0345  0.0264  0.0585  0.0799  0.0134 -0.0087 -0.0053     -0.0302 -0.0020 -0.0717  0.0976 -0.2104  0.0094 -0.0829
  0.0033  0.0051  0.0809 -0.1147  0.2741  0.0162 -0.0007      0.0023  0.0210  0.0663  0.0983  0.0165 -0.0316 -0.0056
  0.0022 -0.0025  0.0042 -0.0596  0.0759  0.0278 -0.0237     -0.0168  0.0073  0.0359 -0.0025  0.0836 -0.0030 -0.0266
  0.0827 -0.0068  0.0178 -0.0018 -0.0437 -0.0635  0.2309      0.0904  0.0166  0.0047 -0.0007 -0.0118  0.0544 -0.0147
 -0.0006 -0.0065  0.0030  0.0066 -0.0251 -0.0477  0.1020      0.0713  0.0005  0.0048 -0.0037 -0.0259 -0.0013  0.0837
spin component  4
  0.1086 -0.0001  0.0115 -0.0011 -0.0299 -0.0017  0.0924     -0.0000  0.0079 -0.0058 -0.0025  0.0204  0.0545 -0.1132
 -0.0001  0.1230  0.0030  0.0019  0.0071  0.0021 -0.0093     -0.0079 -0.0000 -0.0026  0.0083 -0.0101 -0.0031 -0.0066
  0.0115  0.0030  0.0625 -0.0078  0.0650 -0.0067  0.0148      0.0058  0.0026 -0.0000  0.0480 -0.0495 -0.0054 -0.0027
 -0.0011  0.0019 -0.0078  0.1029 -0.0894 -0.0185  0.0053      0.0025 -0.0083 -0.0480 -0.0000 -0.0867  0.0057  0.0087
 -0.0299  0.0071  0.0650 -0.0894  0.2707  0.0281 -0.0766     -0.0204  0.0101  0.0495  0.0867  0.0000 -0.0489  0.0163
 -0.0017  0.0021 -0.0067 -0.0185  0.0281  0.0682 -0.0829     -0.0545  0.0031  0.0054 -0.0057  0.0489  0.0000 -0.0621
  0.0924 -0.0093  0.0148  0.0053 -0.0766 -0.0829  0.3109      0.1132  0.0066  0.0027 -0.0087 -0.0163  0.0621 -0.0000
 
2 3 4                           
spin component  1
  0.2942 -0.0147  0.0789  0.0078 -0.0297 -0.0015  0.0876     -0.0000 -0.0006  0.0021  0.0039 -0.0215 -0.2219  0.0922
 -0.0147  0.1290 -0.0137 -0.0186  0.0035 -0.0020 -0.0045      0.0006  0.0000 -0.0188  0.0116 -0.0147  0.0165 -0.0121
  0.0789 -0.0137  0.3010  0.0060  0.0652  0.0030  0.0285     -0.0021  0.0188  0.0000 -0.2886  0.0733 -0.0890  0.0242
  0.0078 -0.0186  0.0060  0.3732 -0.0933  0.0226  0.0062     -0.0039 -0.0116  0.2886  0.0000  0.0864 -0.0013  0.0048
 -0.0297  0.0035  0.0652 -0.0933  0.0832  0.0134 -0.0212      0.0215  0.0147 -0.0733 -0.0864 -0.0000  0.0168 -0.0044
 -0.0015 -0.0020  0.0030  0.0226  0.0134  0.2214 -0.0798      0.2219 -0.0165  0.0890  0.0013 -0.0168 -0.0000  0.0738
  0.0876 -0.0045  0.0285  0.0062 -0.0212 -0.0798  0.0945     -0.0922  0.0121 -0.0242 -0.0048  0.0044 -0.0738 -0.0000
spin component  2
 -0.1042  0.0116 -0.0345 -0.0033  0.0022 -0.0827 -0.0006     -0.0966  0.0055 -0.0302  0.0023  0.0168  0.0904 -0.0713
  0.0049 -0.0041  0.0264  0.0051  0.0025 -0.0068  0.0065     -0.0083  0.0040  0.0020 -0.0210  0.0073 -0.0166  0.0005
 -0.0015  0.0017 -0.0585 -0.0809  0.0042 -0.0178  0.0030     -0.0130  0.0001 -0.0717  0.0663 -0.0359  0.0047 -0.0048
  0.0115 -0.0043 -0.0799  0.1147 -0.0596  0.0018  0.0066     -0.0060 -0.0097  0.0976  0.0983  0.0025 -0.0007  0.0037
 -0.0095 -0.0152  0.0134  0.2741 -0.0759 -0.0437  0.0251     -0.0758 -0.0038  0.2104 -0.0165  0.0836  0.0118 -0.0259
 -0.0673  0.0033  0.0087 -0.0162  0.0278  0.0635 -0.0477      0.0757  0.0048  0.0094 -0.0316  0.0030  0.0544  0.0013
 -0.0186  0.0002 -0.0053 -0.0007  0.0237  0.2309 -0.1020      0.2714 -0.0163  0.0829  0.0056 -0.0266  0.0147  0.0837
spin component  3
 -0.1042  0.0049 -0.0015  0.0115 -0.0095 -0.0673 -0.0186      0.0966  0.0083  0.0130  0.0060  0.0758 -0.0757 -0.2714
  0.0116 -0.0041  0.0017 -0.0043 -0.0152  0.0033  0.0002     -0.0055 -0.0040 -0.0001  0.0097  0.0038 -0.0048  0.0163
 -0.0345  0.0264 -0.0585 -0.0799  0.0134  0.0087 -0.0053      0.0302 -0.0020  0.0717 -0.0976 -0.2104 -0.0094 -0.0829
 -0.0033  0.0051 -0.0809  0.1147  0.2741 -0.0162 -0.0007     -0.0023  0.0210 -0.0663 -0.0983  0.0165  0.0316 -0.0056
  0.0022  0.0025  0.0042 -0.0596 -0.0759  0.0278  0.0237     -0.0168 -0.0073  0.0359 -0.0025 -0.0836 -0.0030  0.0266
 -0.0827 -0.0068 -0.0178  0.0018 -0.0437  0.0635  0.2309     -0.0904  0.0166 -0.0047  0.0007 -0.0118 -0.0544 -0.0147
 -0.0006  0.0065  0.0030  0.0066  0.0251 -0.0477 -0.1020      0.0713 -0.0005  0.0048 -0.0037  0.0259 -0.0013 -0.0837
spin component  4
  0.1086  0.0001  0.0115 -0.0011  0.0299 -0.0017 -0.0924     -0.0000 -0.0079 -0.0058 -0.0025 -0.0204  0.0545  0.1132
  0.0001  0.1230 -0.0030 -0.0019  0.0071 -0.0021 -0.0093      0.0079 -0.0000  0.0026 -0.0083 -0.0101  0.0031 -0.0066
  0.0115 -0.0030  0.0625 -0.0078 -0.0650 -0.0067 -0.0148      0.0058 -0.0026 -0.0000  0.0480  0.0495 -0.0054  0.0027
 -0.0011 -0.0019 -0.0078  0.1029  0.0894 -0.0185 -0.0053      0.0025  0.0083 -0.0480 -0.0000  0.0867  0.0057 -0.0087
  0.0299  0.0071 -0.0650  0.0894  0.2707 -0.0281 -0.0766      0.0204  0.0101 -0.0495 -0.0867  0.0000  0.0489  0.0163
 -0.0017 -0.0021 -0.0067 -0.0185 -0.0281  0.0682  0.0829     -0.0545 -0.0031  0.0054 -0.0057 -0.0489  0.0000  0.0621
 -0.0924 -0.0093 -0.0148 -0.0053 -0.0766  0.0829  0.3109     -0.1132  0.0066 -0.0027  0.0087 -0.0163 -0.0621 -0.0000
 
3 3 4 
spin component  1
  0.1086  0.0001  0.0115  0.0011 -0.0299  0.0017  0.0924     -0.0000 -0.0079 -0.0058  0.0025  0.0204 -0.0545 -0.1132
  0.0001  0.1230 -0.0030  0.0019 -0.0071  0.0021  0.0093      0.0079 -0.0000  0.0026  0.0083  0.0101 -0.0031  0.0066
  0.0115 -0.0030  0.0625  0.0078  0.0650  0.0067  0.0148      0.0058 -0.0026 -0.0000 -0.0480 -0.0495  0.0054 -0.0027
  0.0011  0.0019  0.0078  0.1029  0.0894 -0.0185 -0.0053     -0.0025 -0.0083  0.0480 -0.0000  0.0867  0.0057 -0.0087
 -0.0299 -0.0071  0.0650  0.0894  0.2707 -0.0281 -0.0766     -0.0204 -0.0101  0.0495 -0.0867  0.0000  0.0489  0.0163
  0.0017  0.0021  0.0067 -0.0185 -0.0281  0.0682  0.0829      0.0545  0.0031 -0.0054 -0.0057 -0.0489  0.0000  0.0621
  0.0924  0.0093  0.0148 -0.0053 -0.0766  0.0829  0.3109      0.1132 -0.0066  0.0027  0.0087 -0.0163 -0.0621 -0.0000
spin component  2
  0.1042 -0.0049  0.0015  0.0115 -0.0095 -0.0673 -0.0186     -0.0966 -0.0083 -0.0130  0.0060  0.0758 -0.0757 -0.2714
 -0.0116  0.0041 -0.0017 -0.0043 -0.0152  0.0033  0.0002      0.0055  0.0040  0.0001  0.0097  0.0038 -0.0048  0.0163
  0.0345 -0.0264  0.0585 -0.0799  0.0134  0.0087 -0.0053     -0.0302  0.0020 -0.0717 -0.0976 -0.2104 -0.0094 -0.0829
 -0.0033  0.0051 -0.0809 -0.1147 -0.2741  0.0162  0.0007     -0.0023  0.0210 -0.0663  0.0983 -0.0165 -0.0316  0.0056
  0.0022  0.0025  0.0042  0.0596  0.0759 -0.0278 -0.0237     -0.0168 -0.0073  0.0359  0.0025  0.0836  0.0030 -0.0266
 -0.0827 -0.0068 -0.0178 -0.0018  0.0437 -0.0635 -0.2309     -0.0904  0.0166 -0.0047 -0.0007  0.0118  0.0544  0.0147
 -0.0006  0.0065  0.0030 -0.0066 -0.0251  0.0477  0.1020      0.0713 -0.0005  0.0048  0.0037 -0.0259  0.0013  0.0837
spin component  3
  0.1042 -0.0116  0.0345 -0.0033  0.0022 -0.0827 -0.0006      0.0966 -0.0055  0.0302  0.0023  0.0168  0.0904 -0.0713
 -0.0049  0.0041 -0.0264  0.0051  0.0025 -0.0068  0.0065      0.0083 -0.0040 -0.0020 -0.0210  0.0073 -0.0166  0.0005
  0.0015 -0.0017  0.0585 -0.0809  0.0042 -0.0178  0.0030      0.0130 -0.0001  0.0717  0.0663 -0.0359  0.0047 -0.0048
  0.0115 -0.0043 -0.0799 -0.1147  0.0596 -0.0018 -0.0066     -0.0060 -0.0097  0.0976 -0.0983 -0.0025  0.0007 -0.0037
 -0.0095 -0.0152  0.0134 -0.2741  0.0759  0.0437 -0.0251     -0.0758 -0.0038  0.2104  0.0165 -0.0836 -0.0118  0.0259
 -0.0673  0.0033  0.0087  0.0162 -0.0278 -0.0635  0.0477      0.0757  0.0048  0.0094  0.0316 -0.0030 -0.0544 -0.0013
 -0.0186  0.0002 -0.0053  0.0007 -0.0237 -0.2309  0.1020      0.2714 -0.0163  0.0829 -0.0056  0.0266 -0.0147 -0.0837
spin component  4
  0.2942 -0.0147  0.0789 -0.0078  0.0297  0.0015 -0.0876     -0.0000 -0.0006  0.0021 -0.0039  0.0215  0.2219 -0.0922
 -0.0147  0.1290 -0.0137  0.0186 -0.0035  0.0020  0.0045      0.0006  0.0000 -0.0188 -0.0116  0.0147 -0.0165  0.0121
  0.0789 -0.0137  0.3010 -0.0060 -0.0652 -0.0030 -0.0285     -0.0021  0.0188 -0.0000  0.2886 -0.0733  0.0890 -0.0242
 -0.0078  0.0186 -0.0060  0.3732 -0.0933  0.0226  0.0062      0.0039  0.0116 -0.2886  0.0000  0.0864 -0.0013  0.0048
  0.0297 -0.0035 -0.0652 -0.0933  0.0832  0.0134 -0.0212     -0.0215 -0.0147  0.0733 -0.0864  0.0000  0.0168 -0.0044
  0.0015  0.0020 -0.0030  0.0226  0.0134  0.2214 -0.0798     -0.2219  0.0165 -0.0890  0.0013 -0.0168 -0.0000  0.0738
 -0.0876  0.0045 -0.0285  0.0062 -0.0212 -0.0798  0.0945      0.0922 -0.0121  0.0242 -0.0048  0.0044 -0.0738 -0.0000
 
4 3 4  
spin component  1
  0.1086 -0.0001  0.0115  0.0011  0.0299  0.0017 -0.0924     -0.0000  0.0079 -0.0058  0.0025 -0.0204 -0.0545  0.1132
 -0.0001  0.1230  0.0030 -0.0019 -0.0071 -0.0021  0.0093     -0.0079 -0.0000 -0.0026 -0.0083  0.0101  0.0031  0.0066
  0.0115  0.0030  0.0625  0.0078 -0.0650  0.0067 -0.0148      0.0058  0.0026 -0.0000 -0.0480  0.0495  0.0054  0.0027
  0.0011 -0.0019  0.0078  0.1029 -0.0894 -0.0185  0.0053     -0.0025  0.0083  0.0480 -0.0000 -0.0867  0.0057  0.0087
  0.0299 -0.0071 -0.0650 -0.0894  0.2707  0.0281 -0.0766      0.0204 -0.0101 -0.0495  0.0867  0.0000 -0.0489  0.0163
  0.0017 -0.0021  0.0067 -0.0185  0.0281  0.0682 -0.0829      0.0545 -0.0031 -0.0054 -0.0057  0.0489  0.0000 -0.0621
 -0.0924  0.0093 -0.0148  0.0053 -0.0766 -0.0829  0.3109     -0.1132 -0.0066 -0.0027 -0.0087 -0.0163  0.0621 -0.0000
spin component  2
 -0.1042 -0.0049 -0.0015 -0.0115 -0.0095  0.0673 -0.0186      0.0966 -0.0083  0.0130 -0.0060  0.0758  0.0757 -0.2714
 -0.0116 -0.0041 -0.0017 -0.0043  0.0152  0.0033 -0.0002      0.0055 -0.0040  0.0001  0.0097 -0.0038 -0.0048 -0.0163
 -0.0345 -0.0264 -0.0585  0.0799  0.0134 -0.0087 -0.0053      0.0302  0.0020  0.0717  0.0976 -0.2104  0.0094 -0.0829
  0.0033  0.0051  0.0809  0.1147 -0.2741 -0.0162  0.0007      0.0023  0.0210  0.0663 -0.0983 -0.0165  0.0316  0.0056
  0.0022 -0.0025  0.0042  0.0596 -0.0759 -0.0278  0.0237     -0.0168  0.0073  0.0359  0.0025 -0.0836  0.0030  0.0266
  0.0827 -0.0068  0.0178  0.0018  0.0437  0.0635 -0.2309      0.0904  0.0166  0.0047  0.0007  0.0118 -0.0544  0.0147
 -0.0006 -0.0065  0.0030 -0.0066  0.0251  0.0477 -0.1020      0.0713  0.0005  0.0048  0.0037  0.0259  0.0013 -0.0837
spin component  3
 -0.1042 -0.0116 -0.0345  0.0033  0.0022  0.0827 -0.0006     -0.0966 -0.0055 -0.0302 -0.0023  0.0168 -0.0904 -0.0713
 -0.0049 -0.0041 -0.0264  0.0051 -0.0025 -0.0068 -0.0065      0.0083  0.0040 -0.0020 -0.0210 -0.0073 -0.0166 -0.0005
 -0.0015 -0.0017 -0.0585  0.0809  0.0042  0.0178  0.0030     -0.0130 -0.0001 -0.0717 -0.0663 -0.0359 -0.0047 -0.0048
 -0.0115 -0.0043  0.0799  0.1147  0.0596  0.0018 -0.0066      0.0060 -0.0097 -0.0976  0.0983 -0.0025 -0.0007 -0.0037
 -0.0095  0.0152  0.0134 -0.2741 -0.0759  0.0437  0.0251     -0.0758  0.0038  0.2104  0.0165  0.0836 -0.0118 -0.0259
  0.0673  0.0033 -0.0087 -0.0162 -0.0278  0.0635  0.0477     -0.0757  0.0048 -0.0094 -0.0316 -0.0030  0.0544 -0.0013
 -0.0186 -0.0002 -0.0053  0.0007  0.0237 -0.2309 -0.1020      0.2714  0.0163  0.0829 -0.0056 -0.0266 -0.0147  0.0837
spin component  4
  0.2942  0.0147  0.0789 -0.0078 -0.0297  0.0015  0.0876     -0.0000  0.0006  0.0021 -0.0039 -0.0215  0.2219  0.0922
  0.0147  0.1290  0.0137 -0.0186 -0.0035 -0.0020  0.0045     -0.0006  0.0000  0.0188  0.0116  0.0147  0.0165  0.0121
  0.0789  0.0137  0.3010 -0.0060  0.0652 -0.0030  0.0285     -0.0021 -0.0188 -0.0000  0.2886  0.0733  0.0890  0.0242
 -0.0078 -0.0186 -0.0060  0.3732  0.0933  0.0226 -0.0062      0.0039 -0.0116 -0.2886  0.0000 -0.0864 -0.0013 -0.0048
 -0.0297 -0.0035  0.0652  0.0933  0.0832 -0.0134 -0.0212      0.0215 -0.0147 -0.0733  0.0864  0.0000 -0.0168 -0.0044
  0.0015 -0.0020 -0.0030  0.0226 -0.0134  0.2214  0.0798     -0.2219 -0.0165 -0.0890  0.0013  0.0168 -0.0000 -0.0738
  0.0876  0.0045  0.0285 -0.0062 -0.0212  0.0798  0.0945     -0.0922 -0.0121 -0.0242  0.0048  0.0044  0.0738 -0.0000

```

## How to cite

For the method please cite the following paper in any publications arising from the use of this code:

Allen J.P. and Watson G.W.,
Occupation matrix control of *d*- and *f*-electron localisations using DFT + *U*, [Physical Chemistry Chemical Physics 16, 21016 (2014)](https://pubs.rsc.org/en/content/articlelanding/2014/cp/c4cp01083c#!divAbstract)

## More
More details can be found in the pdf in the directory for each version.

## License
  This project is licensed under the MIT License - see the [LICENSE.md](./LICENSE.md) for details











