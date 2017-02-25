# Atomic_Subroutines--Using_Coarray_Arrays_to_Allow_for_Safe_Remote_Communication
Fortran 2008 coarray programming with unordered execution segments (user-defined ordering) - Atomic Subroutines: Using Coarray Arrays to Allow for Safe Remote Communication

# Overview
This GitHub repository aims to show a simple coarray programming technique, for use with atomic subroutines, that prevents that coarray values are getting overwritten by other coarray images (i.e. remote processes): The use of coarray arrays does allow for safe communication among a number of coarray images. 

# Problem and Solution
One problem with remote coarray communication is that, if multiple coarray images (say images 2, 3, 4) require to transmit a value to the same remote image (say image 1) through the same scalar coarray variable, a coarray image (say image 2) will probably overwrite the already transmitted values from the other images (3 and 4) before they where consumed by the remote image (1). To prevent this from happening, we may use coarray arrays to remotely transmit scalar coarray values: Every coarray image (2, 3, 4) then uses it's own coarray array index as it's own unique communication channel to a single coarray image (1). Then, the remote coarray image (1) can consume the transmitted values each safely through it's own coarray array index (2, 3, 4).
That programming technique is supported for atomic subroutines by the compilers (ifort, OpenCoarrays/gfortran): While atomic subroutines do only allow to transmit single scalar values, the compilers do still allow to use coarray arrays (as well as array components of derived type coarrays – ifort only with the later compilers, ifort 15 did not work for this) with the atomic subroutines atomic_define and atomic_ref.

The following code snippets show this basically:

Firstly, we declare a derived type containing an atomic integer array, with dimension (1:NumImages), where NumImages shall be a global constant (Parameter), containing a max number of images:

type, public :: ImageStatus_CA<br />
&nbsp;&nbsp;private<br />
&nbsp;&nbsp;!<br />
&nbsp;&nbsp;integer(atomic_int_kind), dimension (1:NumImages) :: mA_atomic_intImageActivityFlag ! mA is short for member array<br />
&nbsp;&nbsp;!<br />
end type ImageStatus_CA<br />

Next, we declare a coarray object from that type:<br />
type (ImageStatus_CA), public, codimension[*], save :: ImageStatus_CA_Object<br />

With that, we can use an intImageNumber variable for both purposes: (1) as the usual cosubscript within square brackets for accessing a coarray object on another image: 'ImageStatus_CA_Object [intImageNumber] %...', and (2), at the same time, as a normal array subscript to access the unique communication channel to that remote image: '...% mA_atomic_intImageActivityFlag(intImageNumber)'. See the following calls to atomic_define and atomic_ref:

call atomic_define(ImageStatus_CA_Object [intImageNumber] % mA_atomic_intImageActivityFlag(intImageNumber), intImageActivityFlag)<br />

call atomic_ref(intImageActivityFlag, ImageStatus_CA_Object %   mA_atomic_intImageActivityFlag(intImageNumber))<br />

As far as to my current knowledge, this simple technique provides a safe way to prevent that atomic values are getting overwritten by other remote processes with there call to atomic_define and thus, that the atomic values can safely be consumed locally by the atomic_ref atomic subroutine.
