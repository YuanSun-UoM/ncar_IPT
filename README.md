# used in Archer2

## FNNL

### build the fire_emis tool

```
cd /work/n02/n02/yuansun/privatemodules_packages/IPT/Emissions/Fire/FINN/src
# do not need to set 'make_fire_emis', make the Makefile directly
make -f Makefile

# need to modify Makefile: add gcc and netcdf path
# if compile the F90 files failed, need to compile one by one manually
# if build mannually, run make -f Makefile to generate the 'fire_emis' exe file finally
make -f Makefile

# convert files from txt to netcdf
# takes some minutes
cd /work/n02/n02/yuansun/privatemodules_packages/IPT/Emissions/Fire/FINN/run
../src/fire_emis < my_finn2_mozT1_camfv.inp > output2018_log.txt 2>&1
# need to modify my_finn2_mozT1_camfv.inp for targtedt data

# move the output to other directory
mv emissions* /work/n02/n02/yuansun/Dataset/NCAR_RDA/d312009/2019_eachfire_modisviirs/fire_emis_0.9_1.25
mv emissions* /work/n02/n02/yuansun/Dataset/NCAR_RDA/d312009/2018_eachfire_modisviirs/fire_emis_0.9_1.25
```

### Error
**error1**: 
(myvenv) (base) yuansun@ln01:/work/n02/n02/yuansun/privatemodules_packages/IPT/Emissions/Fire/FINN/src> make fire_emis
f77 -g -c -I/include attr_types.f90
make: f77: Command not found
make: *** [Makefile:62: attr_types.o] Error 127
**solved**: need to specify FC path:

```
# approach1
make -f Makefile FC=/opt/cray/pe/gcc/11.2.0/bin/gfortran

# approach2
add 'FC=/opt/cray/pe/gcc/11.2.0/bin/gfortran' to Makefile
```

 

**error2**:

```
 /opt/cray/pe/gcc/11.2.0/bin/gfortran -g -c -I/opt/cray/pe/netcdf-hdf5parallel/4.9.0.1/gnu/9.1/include camse_utils.f90
camse_utils.f90:386:132:

  386 |        write(*,'(''camse_mapper: fire @ (lon,lat) = ('',1pg15.7,'','',g15.7,'') not found; firendx = '',i6)') fireLon,fireLat,fireNdx
      |                                                                                                                                    1
Error: Line truncated at (1) [-Werror=line-truncation]
camse_utils.f90:386:132:

  386 |        write(*,'(''camse_mapper: fire @ (lon,lat) = ('',1pg15.7,'','',g15.7,'') not found; firendx = '',i6)') fireLon,fireLat,fireNdx
      |                                                                                                                                    1
Error: Symbol 'firend' at (1) has no IMPLICIT type; did you mean 'firendx'?
f951: some warnings being treated as errors
make: *** [Makefile.std:62: camse_utils.o] Error 1
```
**solved**: the code is too long, change to two lines



**error3**:

```
/opt/cray/pe/gcc/11.2.0/bin/gfortran -g -c -I/opt/cray/pe/netcdf-hdf5parallel/4.9.0.1/gnu/9.1/include wrf_utils.f90
wrf_utils.f90:704:67:

  704 |    call handle_ncerr( nf_get_att_int( ncid, nf_global, 'MAP_PROJ', proj%code ), message )
      |                                                                   1
......
  850 |          call handle_ncerr( nf_get_att_int( ncid, nf_global, trim(attr_name), attrs(m)%attr_int ), message )
      |                                                                              2
Error: Rank mismatch between actual argument at (1) and actual argument at (2) (rank-1 and scalar)
```

**solved**: ref: https://forum.mmm.ucar.edu/threads/how-to-fix-rank-mismatch-between-actual-argument-at-1-and-actual-argument-at-2-scalar-and-rank-1.14995/, add 
export FFLAGS="-fallow-argument-mismatch -fallow-invalid-boz"
export FCFLAGS="-fallow-argument-mismatch"
but not work. In the end, I build the f90 manually:

```
/opt/cray/pe/gcc/default/bin/gfortran -g -c -I/opt/cray/pe/netcdf/4.9.0.1/gnu/9.1/include wrf_utils.f90 -fallow-argument-mismatch -fallow-invalid-boz
```

 

**error**:

```
(myvenv) (base) yuansun@ln01:/work/n02/n02/yuansun/privatemodules_packages/IPT/Emissions/Fire/FINN/src> make -f Makefile.std FC=/opt/cray/pe/gcc/11.2.0/bin/gfortran
m2c    -o wrf_utils.o wrf_utils.mod
make: m2c: Command not found
make: *** [<builtin>: wrf_utils.o] Error 127
```
**solved**: delete already build files before a new build

```
rm -rf *.o *.mod
```

 
