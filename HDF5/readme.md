# Description
HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

**Below are reported vulnerabilities along with Internal Jira bug ID.**

* Heap Buffer Overflow in h5stat 1.10.5 - [HDFFV-10921](https://github.com/fuzzme-ops/Fuzzing-POC/blob/master/HDF5/HDFFV-10921.md) 
* h5dump (1.10.5) SEGV in function H5T_vlen_reclaim_recurse - [HDFFV-10922](https://github.com/fuzzme-ops/Fuzzing-POC/blob/master/HDF5/HDFFV-10922.md)
* Heap Buffer Overflow in h5dump 1.10.5 H5HG_read - [HDFFV-10293](https://github.com/fuzzme-ops/Fuzzing-POC/blob/master/HDF5/HDFFV-10293.md)
* Heap Buffer Overflow in h5ls 1.10 - [HDFFV-10924](https://github.com/fuzzme-ops/Fuzzing-POC/blob/master/HDF5/HDFFV-10924.md)
* h5repack (1.10.5) SEGV in function H5F__close - [HDFFV-10926](https://github.com/fuzzme-ops/Fuzzing-POC/blob/master/HDF5/HDFFV-10926.md)
* h5repack (1.10.5) SEGV in function H5T_close_real - [HDFFV-10927](https://github.com/fuzzme-ops/Fuzzing-POC/blob/master/HDF5/HDFFV-10927.md)
* h5repack (1.10.5) Double Free in function H5MM_xfree - [HDFFV-10928](https://github.com/fuzzme-ops/Fuzzing-POC/blob/master/HDF5/HDFFV-10928.md)

## All above issues are reported on 3/4-Oct-2019.
