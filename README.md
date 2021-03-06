# MacroBuilder 

by E. M. del Pico, S. Ozkan and A. Fernández

## User Manual

### Table of contents


1. [Introduction](#introduction)

2. [Requirements](#requirements)

3. [Installation](#installation)

4. [Why MacroBulding](#why)

5. [How MacroBuilder works](#how)
    1. [Workflow](#workflow)
    2. [Parsing of input and preparation of structures](#pars)
    3. [Main algorithm](#algorithm)
    4. [Structural information](#struct_info)
    3. [Module structure](#structure)

6. [Usage](#usage)
    1. [Arguments](#arguments)
    2. [Input](#input)
       1. Pair Input file
       2. Atomic 3D structure of pairs
    3. [Output](#output)
       1. Report
       2. Final Macrocomplex and Intermediates
       3. Fixed atomic 3D structure files
       4. Distance Plots for intermediates and final complexes
       5. HTML results
       6. Translation and rotation matrices

7. [Examples](#examples)
    1. [Example 1: a simple run.](#ex1)
    2. [Example 2: chain name redundancy.](#ex2)
    3. [Example 3: construction of a multimere from a single homodimer.](#ex3)
    4. [Example 4: more than one molecular type in one complex (DNA-protein).](#ex4)
    5. [Example 5: pairs of non-identical structures.](#ex5)

## 1. Introduction <a name="introduction"></a>

__MacroBuilder__ is a program that builds a macrocomplex given a set of pairs of 3D atomic structure. Construction of the macroxomplex is a sequence of steps, each of which consists in the addition of a new structure to the growing complex. In order to do so, one of the structures in the new pair is superimposed with an already existing structure in the growing complex to correctly position the other structure, that will be actually added to the complex. For a set of pairs with same and/or different molecule types, either in PDB or mmCIF file formats, in adittion to the final complex atomic structure, Macrobuilder generates the intermediate complexes and other useful structural information (e.g. RMSD, distance plot).

## 2. Requiremments <a name="requirements"></a>

MacroBuilder is written in Python (developed and tested in Python 3.5) and requires the packages listed below.

|Package      | Tested version  | About       |
|-------------|-----------------|-------------|
|Biopython    | 1.70            | http://biopython.org |
|Scipy        | 1.0.0           | https://scipy.org/ |
|Numpy        | 1.14.1          | https://docs.scipy.org/doc/numpy/
|Matplotlib   | 1.5.1           |  https://matplotlib.org/|

## 3. Installation <a name="installation"></a>
#### Unix based systems:

##### As program
In order to use MacroBuilder as a program, [download](http://github.com/EvaMart/MacroBuilder/blob/master/MacroBuilder_1.0.tar.gz) and uncompress the package in the desired directory. 
It is ready to use.


##### As a package 
To install MacroBuilder as a package: 

 1- [Download](http://github.com/EvaMart/MacroBuilder/blob/master/macrobuilder.tar.gz) and uncompress the package in the desired directory.
 2- In the same directory, run the following command in the terminal:
```
pip install .
```
To check if correctly installed, you can open a python3 terminal and introduce:
```
>>> import macrobuilder
>>> macrobuilder.test()
``` 
It should print:
```
You are using a MacroBuilder function.
``` 


## 4. Why MacroBuilder <a name="why"></a>

 
MacroBuilder is a tool that performs the construction of complex from pairs of structures. Although simple, this task can be potentially very useful in a wide range of scenarios. 
In the context of macromolecular tertiary and quaternary structure study, atomic resolution structures from X-ray crystallography are not always available for multiple reasons: lack of resources, experimental impossibility, etc. MacroBuilder allows to build hypothetical macrocomplexes containing as many protein as we want as long as superimposition (*) of part of the structures provided is possible.  
MacroBuilder enables the construction of hypothetical quimera complexes, where the molecules in an original complex are substituted by same molecules from a different source, having been crystallized in different conditions, or having been mutated (provided that we have their structure interacting with - or relative to - at least one other member of the complex). Also, new partners can be added to a complex if we have the structure that result from its interaction with one member of a complex. In the same manner, MacroBuilder can also easily build homomultimers from one single structure of a homodimer. 

(*) Superimposition is possible for molecules of same lenght and type.


## 5. How MacroBuilder works <a name="how"></a>

### 5.1. Workflow <a name="workflow"></a>

![](Images/workflow.png)

### 5.1. Parsing of input and preparation of structures.<a name="pars"></a>

MacroBuilder starts by **parsing** the input file; that contain the information relative to the pairs (structures ids, chain-identifiers, molecular type and structure filepath). Then, it **checks** if there is any **inconsistency** (chain-identifier of same structure given differently in different pairs) or **redundancy** (same chain-identifier used for more than one structure) in chain-identifiers, given that for superimposition, same, and only same, chains must share identifier. If detected any; new chain-identifiers are assigned to these chains, and saved as a new file (with .fixed extension). The algorithm makes sure that new assigned chain-identifiers are different from already existing pairs in the input. If the input file contains many pairs with same chain-identifiers, or if a multimer is aimed to be constructed out of homodimers, it is advantageous to change all the chain-identifiers of the whole set (use ***-multi*** flag as an optional argument). 
 The program terminates if all possible letters are used for reassigning the chain-identifiers. The **.pdb** file format allows only one character (**55 unique** possible ids) for chain-identifiers whereas .mmcif files support up to two letters. Considering the number of combinations with two letters being numerically more abundant than those with one letter; very large macromolecules compulsorily constructed in **.mmcif** (**3969 unique** psossible ids) file format.

### 5.2. Main algorithm <a name="algorithm"></a>
MacroBuilder builds the final complex in an iterative manner: through a sequence of discrete steps in which one new structure is added to a growing complex. 
Each iteration consists in three main operations:

 1. **Superimposition.**  
 
 After parsing (and fixing the pairs if required), pairs are put in sequence to make sure that the pairs selected from the sequence has a common structure with each other therefore can be superimposed. First, the algorithm selects the first two sequences to start superimposition. MacroBuilder superimposes structures using the typical approach of rotating and translating them to minimize the root-mean-square-deviation (RMSD), which is the arithmetic average of the coordinate differences between corresponding atoms of the structures. 
     Depending on the molecule type; it gets the coordinates of a subset of atoms of both of the pairs (CA atoms for proteins, C<sub>3</sub>, C<sub>4</sub>, C<sub>5</sub>, O<sub>3</sub>, O<sub>5</sub> and P atoms for nucleotides and all atoms if no molecular type is specified). The coordinates are used for calculating rotation, translation matrices and RMSD values. One of the structures is set as *"base"* and the other one as *"newly added structure"* to make sure that the algorithm knows which structure is going to be translated, so that the centers of both structures' geometry are located at the same place.

 2. **Rotation and translation of incoming structure.** 
   
Uncommon chains (chains that were not used for superimposition) from the incoming structure are rotated and translated using the rotation and translation matrices obtained in the previous step.  

 3. **Generation of 3D atomic structure of new intermediate (previous intermediate + incoming structure).**  
 
Uncommon chains from the incoming structure are then merged with the base using the new coordinates; resulting in one complex. The algorithm then selects the following structure from the sequence, where this time the previously generated complex is considered as base and the one coming from the sequence as newpair. This iteration contines until all the structures in the sequence are merged  into the base. 
An output file is generated for complexes generated within each iteration, named *base_n* where *n* is the number of iteration. For a sequence of *N* pairs, therefore, the n<sup>th</sup> bae represents the final complex. 

### 5.3. Structural information <a name="struct_info"></a>

#### Distance matrix 
In addition, in order to obtain more meaningful information about the structure of the bases and to be able to make comparisons, distance matrices are calculated. It contains all square distances between residues in a structure. Within this purpose, the algorithm calculates distance matrices between the base and new chain-identifiers coming from the new pair; the algorithm measures the distance between the atoms of the backbone of the corresponding molecule type. In case the molecular type is not or cannot be specified (e.g. unhomogenous composition), distance matrix is computed considering all the atoms. The distance matrices are further used for generating distance plots to visualize the position of the new chain compared to all other chains of the base structure.

#### Clashes
Clashes are unfavorable interactions of two non-bonding atoms in a protein structure, which are considered as artifacts. Determination of clashes could be critical for testing the protein quality, understanding possible energetic effects on the protein structure which depends on the types of atoms involved in the clash. 
The algorithm calculates the distance between the center of all atoms in different chains for the final complex and compares it to the sum of their radii: if the distances is smaller than the sum of radii by a quantity below the threshold the user introduced, it  they are reported as clashes, and thus listed in the output file as a block.

### 5.4. Module structure <a name="structure"></a>
Modules used in the program are listed below:
*  **pdb_specific.py:** 
  * *used for parsing .pdb files*
* **mmcif_specific.py** 
  * *used for parsing .mmcif files*
* **fixPairs.py** 
  * *used for correction of inconsistency / redundancy of pairs input file*
* **html_templs.py** 
  * *used for generating the .html output*
* **ngl.js** (*) 
  * *used for setting up 3D view in .html output.*
* **elements.py** (*) 
  * *contains information and methods relative to the periodic table. Used for calculating covalent radii of atoms.*

*These scripts were not developed by MacroBuilder context or developers.

## 6. Usage <a name="usage"></a>

### 6.1. Arguments <a name="arguments"></a>

MacroBuilder is a command line program. The command for its execution is of the following form:

```
python3 macrobuilder.py -i <pairs_input_file> -f <format> -o <output_name> [<Options>]

```

#### Options:

|Option                | &nbsp;&nbsp;Argument            | &nbsp;&nbsp;Requiered   | &nbsp;Description |
|:---------------------|:-------------------|:-----------|:-------------|
| `-i/--input`         | &nbsp;&nbsp;`<pairs_input_file>`      | &nbsp;&nbsp;Yes         | &nbsp;This plain text file contains the details about the pairs from which to build the complex (see details below) |
| `-o/--output`        | &nbsp;&nbsp;`<output_name>`     | &nbsp;&nbsp;Yes | &nbsp;Name of the ouput report|
| `-f/--format`        | &nbsp;&nbsp;`<format>`          | &nbsp;&nbsp;Yes | &nbsp;Format of the 3D molecular structures specified inside the `<input file>` that will be used to build the complex. MacroBuilder supports **pdb** (`-f pdb`) and **mmcif** (` -f mmcif `).  |
| `-multi/--multimere` | &nbsp;&nbsp; *none*             | &nbsp;&nbsp;No  | &nbsp;To build a homomultimer from one identical homodimer. MacroBuilder will deal with chain names (which are identical for all the monomer) accordingly.|
| `-c/-clashes`        | &nbsp;&nbsp;`<threshold>`       | &nbsp;&nbsp;No  | &nbsp;To compute the pairwise distances of all the atoms in the intermedites and final complex. If distance between two atoms is greater than the sum of their covalent radii by more than the `<threshold>`, the "clash" will be reported. `<threshold>` unit: Amstrongs.|
| `-mx/--matrix`       | &nbsp;&nbsp;*none*              | &nbsp;&nbsp;No  | &nbsp;To output the transition and rotation matrices used to generate each base. |

### 6.2. Input <a name="input"></a>:

#### 6.2.1. Pairs Input file:
The input file is a plain text in which each line contains information about one pair to be used in the construction of the complex. Each line is of the form:
```
 <struct1 id>   <struct2 id>   <struct1 chain ids>   <struct2 chain ids>   <type struct1>   <type struct1>   <file name>
 ```
where fields are tab-separated. Fields correspond to:

 - `<struct1 id>`: identifier of one of the structures in the pair.
 - `<struct2 id>`: identifier of the other structure in the pair.
 - `<struct1 chain ids>`: identifiers of chain(s) in struct1. If more than one chain, must be separated by commas (e.g. `A,B,C,K`).
 - `<struct2 chain ids>`: identifiers of chain(s) in struct2. If more than one chain, must be separated by commas (e.g. `A,B,C,K`).
 - `<type struct1>`: molecular type of struct1. Three possibilities: `P` for peptidic, `N`for nucleic (RNA or DNA) and `-` for other or mixed.
 - `<type struct1>`: molecular type of struct1. Three possibilities: `P` for peptidic, `N`for nucleic (RNA or DNA) and `-` for other or mixed.
 - `<file name>`: path of the 3D atomic structure file of the pair.


#### 6.2.2. Atomic 3D structure of pairs:
The user must provide the 3D structure of each pair used in the construction of the complex, whose paths are specified in the Pairs Input file. MacroBuilder supports **PDB** and **mmCIF** formats.

### 6.3. Output: <a name="output"></a>

#### 6.3.1. Report:

Plain text containing at least one block of relevant information. An excerpt from example data is shown bellow:
```
# REPORT
FILE		RMSD		MIN.DIS		MAX.DIS		SUPERIMPOSED
base1       0.000		4.819		123.874		pair_his3_sc_XA.fixed - pair_his3_sc_XB.fixed
base2       0.000		4.322		104.753		pair_his3_sc_XA.fixed - pair_his3_sc_XD.fixed
base3    	0.000		7.099		113.350		pair_his3_sc_XA.fixed - pair_his3_sc_XF.fixed
base4      	0.000		30.156		135.783		pair_his3_sc_XA.fixed - pair_his3_sc_XH.fixed
base5       0.000		4.506		134.775		pair_his3_sc_XA.fixed - pair_his3_sc_XJ.fixed
```
 If there are inconsistencies and/or redundancies in the chain identifiers and thus files need to be fixed, a second block containing information relative to this proccess is added before the mentioned one. 
  ```
 # FIXED PAIRS
 FILE                   FIXED_PAIR  CHAIN_ID    NEW_CHAIN_ID    NEWFILE
 pair_his3_sc_XT        T           B           C               pair_his3_sc_XT.fixed.pdb
 pair_his3_sc_XO        O           B           D               pair_his3_sc_XO.fixed.pdb
 pair_his3_sc_XS        S           B           E               pair_his3_sc_XS.fixed.pdb
 pair_his3_sc_XU        U           B           F               pair_his3_sc_XU.fixed.pdb
 pair_his3_sc_XC        C           B           G               pair_his3_sc_XC.fixed.pdb
 ```

 If the user introduced the `-c` option, found clashes will be shown as an additional block at the end of the file. 
 ```
 # CLASHES
 Threshold: 0.01
 ATOM	RES	 CHAIN	ATOM	RES	 CHAIN	DIST	SUM_OF_RADII
 O   	GLY	 A	    C	    ALA	 B	    3.167	3.220
 C	    ALA  B	    O	    GLY	 A  	3.167	3.220
 O	    GLY	 C	    C	    ALA	 D	    3.166	3.220
 O	    GLY	 G	    C	    ALA	 H	    3.167	3.220
 ```

#### 6.3.2. Final Macrocomplex and Intermediates:
The final macrocomplex as well as all the intermediary complexes generated in its construction (bases) are returned in the same format as the 3D atomic structures provided by the user. The files are called *base<N>.<ter>*, being *<N>* the intermediate to which it belong and *<ter>* the appropriate extension (.pdb or .cif).

#### 6.3.3. Fixed atomic 3D structure files:
3D atomic structure files generated when redundancies and/or inconsistencies are detected. They contain the exact same structure as the provided files, except for the chain identifiers, which are changed to ensure consistency and non-redundancy.

#### 6.3.4. Distance Plots for intermediates and final complexes (*finalplot.png* and *baseN.png*):
In each step of the construction, the distance between the previously generated base and the newly added structure is computed and a plot of them is generated. Distance plots are called *baseN.png*, and N being the intermediate to which it belongs. 
A plot of the distances between all the atoms in the final macrocomplex,*finalplot.png*, is also generated.

#### 6.3.5. HTML results (*results.html*):
This *html* file contains the results of a MacroBuilder run in a more visual form. It includes the details about the job, 3D atomic representations of the final complex and its distance plot.
In addition, a list of relevant parameters in each pair addition (RMSD of common chain in pair and minimum and maximum distances between old and newly added atoms) and a link to the rpresentations of 3D structure and plot of the resulting intermediate. 

#### 6.3.6. Translation and rotation matrices:
`matrix.trans` file contains, for each base, the rotation and translation matrices that the main algorithm used to position the last added structure. And excerpt from example data showing the rotation (top) and translation (bottom) matrices for one base. 
```
# P1
[ 0.64022897 -0.74406983 -0.19096323]
[ 0.52858182  0.6070907  -0.59333139]
[ 0.55741199  0.27892825  0.78197884]

[12.20881255 59.56930054 26.69124712]
```


## 7. Examples <a name="examples"></a>
### 7.1. Example 1: a simple run (PDB). <a name="ex1"></a>
We want to build the ATP-synthase from a set of pairs of which we have the 3D atomic structure. The Input Pair file for these pairs looks as follows:
```
 Prot1	Prot2	1,2,3	A	P	P	5dn6123A.pdb
 Prot2	Prot3	A	B,C	P	P	5dn6ABC.pdb
 Prot3	Prot4	B,C	D,E	P	P	5dn6BCDE.pdb
 Prot4	Prot5	D,E	F,G	P	P	5dn6DEFG.pdb
...
```
Pairs do not need to be in a specific order.
To build the complex, without using any option (minimal run), we run:
```
python3 macrobuilder.py -i pairs.txt -o output.txt -f pdb
```
This yields the standard output.

### 7.2. Example 2: chain name redundancy (PDB).<a name="ex2"></a>
We want to build a complex from 23 pairs of proteic structures, each of which in constituted by one chain. The Input Pairs file (*pairs.txt*) looks as follows:
```
 X	A	A	B	P	P	example1/pair_his3_sc_XA.pdb
 X	B	A	B	P	P	example1/pair_his3_sc_XB.pdb
 X	C	A	B	P	P	example1/pair_his3_sc_XC.pdb

 ...
```

In this case, the chain name *B* is repeated for many different proteins. MacroBuilder needs different structures to have distinct chain names, so, in case of detecting a redundancy/inconsistency in the chain naming, it will rewrite the 3D structure files using new non-redundant and consistent chain names. These *fixed* files are have the extension *.fixed.<original extension>* and are store in the working directory. Then, the complex construction will proceed as usual.

To build the complex we run:
```
 python3 macrobuilder.py -i pairs.txt -o report.txt -f pdb -mx
```
Apart from the standard outputs, in this specific case we are also asking for the translation and rotation matrices used to correctly position the structure that is added to each intermidiate complex (`-mx`). 

### 7.3. Example 3: construction of a multimere from a single homodimer (CIF). <a name="ex3"></a>
We want to build an homomultimer using only one homodimer. This is, to build an homomultimer by combining one homodimer with itself several times. In order to do so, we need to make as many copies of the homodimer 3D atomic structure file as monomers (halfs of the dimer) as we want to add and build a Pair Input file like the following one:
```
 A	B	K,L	M,N	P	P	5dn6_KLMN.1.cif
 B	C	K,L	M,N	P	P	5dn6_KLMN.2.cif
 C	D	K,L	M,N	P	P	5dn6_KLMN.3.cif
 D	E	K,L	M,N	P	P	5dn6_KLMN.4.cif
 E	F	K,L	M,N	P	P	5dn6_KLMN.5.cif
```
 All the files have a different name. All the pairs have the same chains, as they are identical, but the structure identifiers are assigned in a way that ensures the construction of the multimer.
 To run MacroBuilder:
```
python3 macrobuilder.py -i pairs.txt -o report.txt -f mmcif -multi -c 0.01

```
The format of the 3D atomic structure files is `mmcif`. The option `-multi` is necessary to change all the chain names from the beginning.
In addition, in this run clashes will be reported if the distance between two atoms is smaller than the sum of their covalent radii by more than 0.01 (`-c 0.01`).

### 7.4. Example 4: more than one molecular type in one complex (DNA-protein). <a name="ex4"></a>
We want to build a complex that contains two molecular types. The Pairs Input file looks as follow:
```
 G	H	G	H	N	N	5wfe_GH.pdb
 H	I	H	I	N	N	5wfe_HI.pdb
 I	J	I	J	N	N	5wfe_IJ.pdb
 K	J	K	J	P	N	5wfe_KJ.pdb
 K	L	K	L	P	P	5wfe_KL.pdb
...
```
To build the complex, we run:

```
python3 macrobuilder.py -i pairs.txt -o report.txt -f pdb
```

### 7.5. Example 5: construction of a complex involving the superposition of non-identical structures. <a name="ex5"></a>
It is also possible to superimpose non-identical structures. In this example we want to reconstruct a human haemoglobin
 that contains both deoxy and carbonoxy forms of the chains. The different forms have a different source but are identical in length. The latter is a neccesary requirement for the construction, otherwise the computation of translation and rotation matrices is not possible due to the impossibility of superimposition.
 The Pairs Input file looks as follow:
 ```
 A	B	A	B	P	P	1hco.pdb
 A	C	A	C	P	P	1a3n_AC.pdb
 A	D	A	D	P	P	1a3n_AD.pdb
 ```
 To build the complex we run:
 ```
  python3 macrobuilder.py -i pairs.txt -o output.txt -f pdb -c 0.4
 ```
 in addition, we ask for clashes with a threshold of 0.4 Amstrongs. 
 Notice the big RMSD in the superimposition of chains A from carbonoxy and deoxy haemoglobins. This is due to conformational differencies.
 
 
 ## 8. Troubleshooting
 
Of all the possible records in a PDB file, only 'ATOM' records are absolutely necessary. 
Other records such as "MODEL" could cause an incorrect parsing and thus make the program crash, so in case of having trouble is advisable to remove any potentially conflictive record types.
 
