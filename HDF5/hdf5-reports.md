# Description
HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

Below are reported vulnerabilities along with Internal Jira bug ID.
```
Heap Buffer Overflow in h5stat 1.10.5  -  HDFFV-10921 
h5dump (1.10.5) SEGV in function H5T_vlen_reclaim_recurse  - HDFFV-10922
Heap Buffer Overflow in h5dump 1.10.5 H5HG_read  - HDFFV-10293
Heap Buffer Overflow in h5ls 1.10.  - HDFFV-10924
h5repack (1.10.5) SEGV in function H5F__close - HDFFV-10926
h5repack (1.10.5) SEGV in function H5T_close_real  - HDFFV-10927
h5repack (1.10.5) Double Free in function H5MM_xfree - HDFFV-10928
```

Lets look at write-up of each bugs.

*** Heap Buffer Overflow in h5stat 1.10.5  -  HDFFV-10921 ***
We found the Heap Buffer Overflow in h5stat 1.10.5 version. I have attached the detailed output of crash and testcase file too.

Compilation steps
```CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```

**ASAN Output**
```fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5stat$ ./h5stat -V
h5stat: Version 1.10.5

fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5stat$ ./h5stat testcase-1570044709-4-8461_h5ex_g_corder.h5
Filename: testcase-1570044709-4-8461_h5ex_g_corder.h5
=================================================================
==5477==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x603000006a78 at pc 0x000000762e06 bp 0x7ffeb37afbd0 sp 0x7ffeb37afbc8
READ of size 1 at 0x603000006a78 thread T0
    #0 0x762e05 in H5F_addr_decode_len /home/fuzzer/victim/hdf5-1.10.5/src/H5Fint.c:2486:13
    #1 0x8cd6fa in H5HL__hdr_deserialize /home/fuzzer/victim/hdf5-1.10.5/src/H5HLcache.c:202:5
    #2 0x8c9310 in H5HL__cache_prefix_get_final_load_size /home/fuzzer/victim/hdf5-1.10.5/src/H5HLcache.c:383:8
    #3 0x5d91c9 in H5C_load_entry /home/fuzzer/victim/hdf5-1.10.5/src/H5C.c:6616:20
    #4 0x5d91c9 in H5C_protect /home/fuzzer/victim/hdf5-1.10.5/src/H5C.c:2340
    #5 0x570fd1 in H5AC_protect /home/fuzzer/victim/hdf5-1.10.5/src/H5AC.c:1351:25
    #6 0x8c8d11 in H5HL_heapsize /home/fuzzer/victim/hdf5-1.10.5/src/H5HL.c:1033:39
    #7 0x836150 in H5G__stab_bh_size /home/fuzzer/victim/hdf5-1.10.5/src/H5Gstab.c:677:8
    #8 0x103d822 in H5O__group_bh_info /home/fuzzer/victim/hdf5-1.10.5/src/H5Goh.c:400:12
    #9 0x9989e6 in H5O_get_info /home/fuzzer/victim/hdf5-1.10.5/src/H5Oint.c:2249:16
    #10 0x81240b in H5G_loc_info_cb /home/fuzzer/victim/hdf5-1.10.5/src/H5Gloc.c:687:8
    #11 0x83e97b in H5G__traverse_real /home/fuzzer/victim/hdf5-1.10.5/src/H5Gtraverse.c:626:16
    #12 0x83bcad in H5G_traverse /home/fuzzer/victim/hdf5-1.10.5/src/H5Gtraverse.c:850:8
    #13 0x81200f in H5G_loc_info /home/fuzzer/victim/hdf5-1.10.5/src/H5Gloc.c:730:8
    #14 0x92ca04 in H5Oget_info_by_name2 /home/fuzzer/victim/hdf5-1.10.5/src/H5O.c:518:8
    #15 0x5327f9 in traverse_cb /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5trav.c:205:12
    #16 0x808349 in H5G_visit_cb /home/fuzzer/victim/hdf5-1.10.5/src/H5Gint.c:951:17
    #17 0x80cf1b in H5G__link_iterate_table /home/fuzzer/victim/hdf5-1.10.5/src/H5Glink.c:484:21
    #18 0x10215c2 in H5G__compact_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gcompact.c:422:21
    #19 0x829161 in H5G__obj_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gobj.c:687:29
    #20 0x80884b in H5G_visit_cb /home/fuzzer/victim/hdf5-1.10.5/src/H5Gint.c:1035:29
    #21 0x81ffcc in H5G__node_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gnode.c:1002:25
    #22 0xfd6ca5 in H5B__iterate_helper /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1165:25
    #23 0xfd66f2 in H5B_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1210:21
    #24 0x835198 in H5G__stab_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gstab.c:556:25
    #25 0x829075 in H5G__obj_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gobj.c:697:25
    #26 0x807136 in H5G_visit /home/fuzzer/victim/hdf5-1.10.5/src/H5Gint.c:1183:21
    #27 0x8f5270 in H5Lvisit_by_name /home/fuzzer/victim/hdf5-1.10.5/src/H5L.c:1297:21
    #28 0x52e3e0 in traverse /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5trav.c:291:16
    #29 0x531a28 in h5trav_visit /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5trav.c:1063:8
    #30 0x4ecd02 in main /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5stat/h5stat.c:1823:16
    #31 0x7fa27463482f in __libc_start_main /build/glibc-LK5gWL/glibc-2.23/csu/../csu/libc-start.c:291
    #32 0x4194e8 in _start (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5stat/h5stat+0x4194e8)

0x603000006a78 is located 0 bytes to the right of 24-byte region [0x603000006a60,0x603000006a78)
allocated by thread T0 here:
    #0 0x4b9618 in malloc (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5stat/h5stat+0x4b9618)
    #1 0x926a79 in H5MM_malloc /home/fuzzer/victim/hdf5-1.10.5/src/H5MM.c:322:21
    #2 0x570fd1 in H5AC_protect /home/fuzzer/victim/hdf5-1.10.5/src/H5AC.c:1351:25

SUMMARY: AddressSanitizer: heap-buffer-overflow /home/fuzzer/victim/hdf5-1.10.5/src/H5Fint.c:2486:13 in H5F_addr_decode_len
Shadow bytes around the buggy address:
  0x0c067fff8cf0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8d00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8d10: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8d20: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8d30: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
=>0x0c067fff8d40: fa fa fa fa fa fa fa fa fa fa fa fa 00 00 00[fa]
  0x0c067fff8d50: fa fa 00 00 00 00 fa fa 00 00 07 fa fa fa 00 00
  0x0c067fff8d60: 07 fa fa fa 00 00 05 fa fa fa 00 00 05 fa fa fa
  0x0c067fff8d70: 00 00 00 fa fa fa 00 00 00 00 fa fa 00 00 00 00
  0x0c067fff8d80: fa fa 00 00 00 00 fa fa 00 00 00 00 fa fa 00 00
  0x0c067fff8d90: 00 00 fa fa 00 00 05 fa fa fa 00 00 05 fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Heap right redzone:      fb
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack partial redzone:   f4
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==5477==ABORTING
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5stat$
```

*** h5dump (1.10.5) SEGV in function H5T_vlen_reclaim_recurse  - HDFFV-10922 ***

We found the h5dump (1.10.5 version) SEGV in function H5T_vlen_reclaim_recurse. I have attached the detailed output of crash and testcase file too.

Compilation steps
```
CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```

**ASAN Output**
```
entropy@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5dump$ ./h5dump -V
h5dump: Version 1.10.5

entropy@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5dump$ ./h5dump -x out/0/crashes/unique/testcase-1570093178-0-2446_h5ex_t_cmpdatt.h5
<?xml version="1.0" encoding="UTF-8"?>
<hdf5:HDF5-File xmlns:hdf5="http://hdfgroup.org/HDF5/XML/schema/HDF5-File.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://hdfgroup.org/HDF5/XML/schema/HDF5-File http://www.hdfgroup.org/HDF5/XML/schema/HDF5-File.xsd">
<hdf5:RootGroup OBJ-XID="xid_96" H5Path="/">
   <hdf5:Dataset Name="DS1" OBJ-XID="xid_800" H5Path= "/DS1" Parents="xid_96" H5ParentPaths="/">
      <hdf5:StorageLayout>
         <hdf5:ContiguousLayout/>
      </hdf5:StorageLayout>
      <hdf5:FillValueInfo FillTime="FillIfSet" AllocationTime="Late">
         <hdf5:FillValue>
            <hdf5:NoFill/>
         </hdf5:FillValue>
      </hdf5:FillValueInfo>
      <hdf5:Dataspace>
         <!-- unknown dataspace -->
      </hdf5:Dataspace>
      <hdf5:DataType>
         <hdf5:AtomicType>
            <hdf5:IntegerType ByteOrder="LE" Sign="true" Size="4" />
         </hdf5:AtomicType>
      </hdf5:DataType>
      <hdf5:Attribute Name="A1">
         <hdf5:Dataspace>
            <hdf5:SimpleDataspace Ndims="1">
               <hdf5:Dimension  DimSize="4" MaxDimSize="4"/>
            </hdf5:SimpleDataspace>
         </hdf5:Dataspace>
         <hdf5:DataType>
            <hdf5:CompoundType>
               <hdf5:Field FieldName="Serial number">
                  <hdf5:DataType>
                     <hdf5:AtomicType>
                        <hdf5:IntegerType ByteOrder="BE" Sign="true" Size="8" />
                     </hdf5:AtomicType>
                  </hdf5:DataType>
               </hdf5:Field>
               <hdf5:Field FieldName="Location">
                  <hdf5:DataType>
                     <hdf5:AtomicType>
                        <hdf5:StringType Cset="H5T_CSET_ASCII" StrSize="H5T_VARIABLE" StrPad="H5T_STR_NULLTERM"/>
                     </hdf5:AtomicType>
                  </hdf5:DataType>
               </hdf5:Field>
               <hdf5:Field FieldName="Temperature (F)">
                  <hdf5:DataType>
                     <hdf5:AtomicType>
                        <hdf5:FloatType ByteOrder="BE" Size="8" SignBitLocation="63" ExponentBits="11" ExponentLocation="52" MantissaBits="52" MantissaLocation="0" />
                     </hdf5:AtomicType>
                  </hdf5:DataType>
               </hdf5:Field>
               <hdf5:Field FieldName="Pressure (inHg)">
                  <hdf5:DataType>
                     <hdf5:AtomicType>
                        <hdf5:FloatType ByteOrder="BE" Size="8" SignBitLocation="63" ExponentBits="11" ExponentLocation="52" MantissaBits="52" MantissaLocation="0" />
                     </hdf5:AtomicType>
                  </hdf5:DataType>
               </hdf5:Field>
            </hdf5:CompoundType>
         </hdf5:DataType>
         <!-- Note: format of compound data not specified -->
         <hdf5:Data>
ASAN:DEADLYSIGNAL
=================================================================
==16620==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x00000041d3ad bp 0x7ffc123501f0 sp 0x7ffc1234f950 T0)
    #0 0x41d3ac in __asan::asan_free(void*, __sanitizer::BufferedStackTrace*, __asan::AllocType) (/home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x41d3ac)
    #1 0x4b96bc in __interceptor_cfree.localalias.0 (/home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x4b96bc)
    #2 0xf63a60 in H5T_vlen_reclaim_recurse /home/entropy/victim/hdf5-1.10.5/src/H5Tvlen.c:1075:21
    #3 0xf632dd in H5T_vlen_reclaim_recurse /home/entropy/victim/hdf5-1.10.5/src/H5Tvlen.c:1038:24
    #4 0xf62ce8 in H5T_vlen_reclaim /home/entropy/victim/hdf5-1.10.5/src/H5Tvlen.c:1149:8
    #5 0xb79c25 in H5S_select_iterate /home/entropy/victim/hdf5-1.10.5/src/H5Sselect.c:1469:36
    #6 0x6d8200 in H5D_vlen_reclaim /home/entropy/victim/hdf5-1.10.5/src/H5Dint.c:2579:17
    #7 0x66575f in H5Dvlen_reclaim /home/entropy/victim/hdf5-1.10.5/src/H5D.c:760:17
    #8 0x5107eb in xml_dump_data /home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:1921:25
    #9 0x513a29 in xml_dump_attr /home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:2048:17
    #10 0x590482 in H5A__attr_iterate_table /home/entropy/victim/hdf5-1.10.5/src/H5Aint.c:1831:29
    #11 0x987d18 in H5O_attr_iterate_real /home/entropy/victim/hdf5-1.10.5/src/H5Oattribute.c:1296:25
    #12 0x98c572 in H5O__attr_iterate /home/entropy/victim/hdf5-1.10.5/src/H5Oattribute.c:1341:21
    #13 0x59781a in H5A__iterate_common /home/entropy/victim/hdf5-1.10.5/src/H5Aint.c:2596:21
    #14 0x59781a in H5A__iterate /home/entropy/victim/hdf5-1.10.5/src/H5Aint.c:2634
    #15 0x57f02a in H5Aiterate2 /home/entropy/victim/hdf5-1.10.5/src/H5A.c:1258:21
    #16 0x520842 in xml_dump_dataset /home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:3922:13
    #17 0x51b90c in xml_dump_all_cb /home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:327:17
    #18 0x84bc27 in H5G_iterate_cb /home/entropy/victim/hdf5-1.10.5/src/H5Gint.c:794:29
    #19 0x8653cc in H5G__node_iterate /home/entropy/victim/hdf5-1.10.5/src/H5Gnode.c:1002:25
    #20 0x1027825 in H5B__iterate_helper /home/entropy/victim/hdf5-1.10.5/src/H5B.c:1165:25
    #21 0x1027272 in H5B_iterate /home/entropy/victim/hdf5-1.10.5/src/H5B.c:1210:21
    #22 0x87a598 in H5G__stab_iterate /home/entropy/victim/hdf5-1.10.5/src/H5Gstab.c:556:25
    #23 0x86e475 in H5G__obj_iterate /home/entropy/victim/hdf5-1.10.5/src/H5Gobj.c:697:25
    #24 0x84b247 in H5G_iterate /home/entropy/victim/hdf5-1.10.5/src/H5Gint.c:855:21
    #25 0x938078 in H5L__iterate /home/entropy/victim/hdf5-1.10.5/src/H5L.c:3185:21
    #26 0x938078 in H5Literate /home/entropy/victim/hdf5-1.10.5/src/H5L.c:1122
    #27 0x517ddf in xml_dump_group /home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:2783:13
    #28 0x4eef75 in main /home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump.c:1564:17
    #29 0x7fc4f6a8b82f in __libc_start_main /build/glibc-LK5gWL/glibc-2.23/csu/../csu/libc-start.c:291
    #30 0x419738 in _start (/home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x419738)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV (/home/entropy/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x41d3ac) in __asan::asan_free(void*, __sanitizer::BufferedStackTrace*, __asan::AllocType)
==16620==ABORTING
entropy@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5dump$
```

*** Heap Buffer Overflow in h5dump 1.10.5 H5HG_read  - HDFFV-10293 ***

We found the Heap Buffer Overflow in h5dump 1.10.5 version. I have attached the detailed output of crash and testcase file too.

Compilation steps
```
CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```
** ASAN Output **
```
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5dump$ ./h5dump -V
h5dump: Version 1.10.5

fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5dump$ ./h5dump -x out/5/crashes/unique/testcase-1570093191-5-2530_h5ex_t_cpxcmpd.h5
<?xml version="1.0" encoding="UTF-8"?>
<hdf5:HDF5-File xmlns:hdf5="http://hdfgroup.org/HDF5/XML/schema/HDF5-File.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://hdfgroup.org/HDF5/XML/schema/HDF5-File http://www.hdfgroup.org/HDF5/XML/schema/HDF5-File.xsd">
<hdf5:RootGroup OBJ-XID="xid_96" H5Path="/">
   <hdf5:Group Name="Air_Vehicles" OBJ-XID="xid_2104" H5Path="/Air_Vehicles" Parents="xid_96" H5ParentPaths="/" >
   </hdf5:Group>
   <hdf5:Dataset Name="Ambient_Temperature" OBJ-XID="xid_800" H5Path= "/Ambient_Temperature" Parents="xid_96" H5ParentPaths="/">
      <hdf5:StorageLayout>
         <hdf5:ContiguousLayout/>
      </hdf5:StorageLayout>
      <hdf5:FillValueInfo FillTime="FillIfSet" AllocationTime="Late">

 ///////SNIPPED/////////////


  <!-- Note: format of compound data not specified -->
      <hdf5:Data>
         <hdf5:DataFromFile>
=================================================================
==16878==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6210005f8908 at pc 0x0000004a359d bp 0x7fff0a9f7fd0 sp 0x7fff0a9f7780
READ of size 3520 at 0x6210005f8908 thread T0
    #0 0x4a359c in __asan_memcpy (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x4a359c)
    #1 0x902473 in H5HG_read /home/fuzzer/victim/hdf5-1.10.5/src/H5HG.c:621:5
    #2 0xb0095e in H5R__dereference /home/fuzzer/victim/hdf5-1.10.5/src/H5Rint.c:418:42
    #3 0xafcbce in H5Rdereference2 /home/fuzzer/victim/hdf5-1.10.5/src/H5R.c:185:21
    #4 0x559214 in h5tools_str_sprint_region /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_str.c:1319:11
    #5 0x555d56 in h5tools_str_sprint /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_str.c:1115:25
    #6 0x5567f0 in h5tools_str_sprint /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_str.c:1077:29
    #7 0x532111 in h5tools_dump_simple_data /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_dump.c:390:17
    #8 0x540653 in h5tools_dump_simple_dset /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_dump.c:1643:12
    #9 0x540653 in h5tools_dump_dset /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_dump.c:1806
    #10 0x510274 in xml_dump_data /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:1864:22
    #11 0x521819 in xml_dump_dataset /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:3992:13
    #12 0x51b90c in xml_dump_all_cb /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:327:17
    #13 0x84bc27 in H5G_iterate_cb /home/fuzzer/victim/hdf5-1.10.5/src/H5Gint.c:794:29
    #14 0x8653cc in H5G__node_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gnode.c:1002:25
    #15 0x1027825 in H5B__iterate_helper /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1165:25
    #16 0x1027272 in H5B_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1210:21
    #17 0x87a598 in H5G__stab_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gstab.c:556:25
    #18 0x86e475 in H5G__obj_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gobj.c:697:25
    #19 0x84b247 in H5G_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gint.c:855:21
    #20 0x938078 in H5L__iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5L.c:3185:21
    #21 0x938078 in H5Literate /home/fuzzer/victim/hdf5-1.10.5/src/H5L.c:1122
    #22 0x517ddf in xml_dump_group /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump_xml.c:2783:13
    #23 0x4eef75 in main /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump.c:1564:17
    #24 0x7ff95101882f in __libc_start_main /build/glibc-LK5gWL/glibc-2.23/csu/../csu/libc-start.c:291
    #25 0x419738 in _start (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x419738)

0x6210005f8908 is located 0 bytes to the right of 4104-byte region [0x6210005f7900,0x6210005f8908)
allocated by thread T0 here:
    #0 0x4b9868 in malloc (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x4b9868)
    #1 0x96be79 in H5MM_malloc /home/fuzzer/victim/hdf5-1.10.5/src/H5MM.c:322:21

SUMMARY: AddressSanitizer: heap-buffer-overflow (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5dump/h5dump+0x4a359c) in __asan_memcpy
Shadow bytes around the buggy address:
  0x0c42800b70d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c42800b70e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c42800b70f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c42800b7100: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c42800b7110: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c42800b7120: 00[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c42800b7130: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c42800b7140: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c42800b7150: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c42800b7160: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c42800b7170: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Heap right redzone:      fb
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack partial redzone:   f4
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==16878==ABORTING
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5dump$
```
*** Heap Buffer Overflow in h5ls 1.10.  - HDFFV-10924 ***

We found the Heap Buffer Overflow in h5ls 1.10.5 version. I have attached the detailed output of crash and testcase file too.

Compilation steps
```
CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```
** ASAN Output ***
```
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5ls$ ./h5ls -V
h5ls: Version 1.10.5

fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5ls$ ./h5ls -rad out/5/crashes/unique/testcase-1570119413-5-9155_h5ex_t_cpxcmpd.h5
/                        Group
/Air_Vehicles            Group
/Ambient_Temperature     Dataset {32, 32}
    Data:
        (0,0) 66.8, 66.9, 67, 67.1, 67.2, 67.3, 67.4, 67.5, 67.6, 67.7, 67.8, 67.9, 68, 68.1, 68.2, 68.3, 68.4, 68.5, 68.6, 68.7, 68.8, 68.9, 69, 69.1, 69.2, 69.3, 69.4, 69.5,
        (0,28) 69.6, 69.7, 69.8, 69.9, 66.9, 67, 67.1, 67.2, 67.3, 67.4, 67.5, 67.6, 67.7, 67.8, 67.9, 68, 68.1, 68.2, 68.3, 68.4, 68.5, 68.6, 68.7, 68.8, 68.9, 69, 69.1, 69.2,

///SNIPPED////

        (30,16) 71.4, 71.5, 71.6, 71.7, 71.8, 71.9, 72, 72.1, 72.2, 72.3, 72.4, 72.5, 72.6, 72.7, 72.8, 72.9, 69.9, 70, 70.1, 70.2, 70.3, 70.4, 70.5, 70.6, 70.7, 70.8, 70.9, 71,
        (31,12) 71.1, 71.2, 71.3, 71.4, 71.5, 71.6, 71.7, 71.8, 71.9, 72, 72.1, 72.2, 72.3, 72.4, 72.5, 72.6, 72.7, 72.8, 72.9, 73
/DS1                     Dataset {2}
    Data:
=================================================================
==5013==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x621000071908 at pc 0x0000004a334d bp 0x7ffc850dd770 sp 0x7ffc850dcf20
READ of size 3520 at 0x621000071908 thread T0
    #0 0x4a334c in __asan_memcpy (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls+0x4a334c)
    #1 0x8c2603 in H5HG_read /home/fuzzer/victim/hdf5-1.10.5/src/H5HG.c:621:5
    #2 0xf0f8ab in H5T_vlen_disk_read /home/fuzzer/victim/hdf5-1.10.5/src/H5Tvlen.c:852:20
    #3 0xc13ac6 in H5T__conv_vlen /home/fuzzer/victim/hdf5-1.10.5/src/H5Tconv.c:3226:32
    #4 0xbd2fab in H5T_convert /home/fuzzer/victim/hdf5-1.10.5/src/H5T.c:5024:12
    #5 0xc0c482 in H5T__conv_struct_opt /home/fuzzer/victim/hdf5-1.10.5/src/H5Tconv.c:2509:28
    #6 0xbd2fab in H5T_convert /home/fuzzer/victim/hdf5-1.10.5/src/H5T.c:5024:12
    #7 0x6c89d2 in H5D__scatgath_read /home/fuzzer/victim/hdf5-1.10.5/src/H5Dscatgath.c:547:16
    #8 0x688662 in H5D__contig_read /home/fuzzer/victim/hdf5-1.10.5/src/H5Dcontig.c:595:8
    #9 0x6b84d3 in H5D__read /home/fuzzer/victim/hdf5-1.10.5/src/H5Dio.c:600:8
    #10 0x6b739f in H5Dread /home/fuzzer/victim/hdf5-1.10.5/src/H5Dio.c:198:9
    #11 0x513d9c in h5tools_dump_simple_dset /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_dump.c:1631:13
    #12 0x513d9c in h5tools_dump_dset /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5tools_dump.c:1806
    #13 0x4efa59 in dump_dataset_values /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls.c:1465:9
    #14 0x4efa59 in dataset_list2 /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls.c:1960
    #15 0x4f8622 in list_obj /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls.c:2143:13
    #16 0x537a80 in traverse_cb /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5trav.c:220:16
    #17 0x80d8d9 in H5G_visit_cb /home/fuzzer/victim/hdf5-1.10.5/src/H5Gint.c:951:17
    #18 0x82555c in H5G__node_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gnode.c:1002:25
    #19 0xfd5c15 in H5B__iterate_helper /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1165:25
    #20 0xfd5662 in H5B_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1210:21
    #21 0x83a728 in H5G__stab_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gstab.c:556:25
    #22 0x82e605 in H5G__obj_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5Gobj.c:697:25
    #23 0x80c6c6 in H5G_visit /home/fuzzer/victim/hdf5-1.10.5/src/H5Gint.c:1183:21
    #24 0x8fa800 in H5Lvisit_by_name /home/fuzzer/victim/hdf5-1.10.5/src/H5L.c:1297:21
    #25 0x5331c0 in traverse /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5trav.c:291:16
    #26 0x536808 in h5trav_visit /home/fuzzer/victim/hdf5-1.10.5/tools/lib/h5trav.c:1063:8
    #27 0x4f1df9 in visit_obj /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls.c:2417:9
    #28 0x4ec8dc in main /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls.c:2908:16
    #29 0x7f64e4eef82f in __libc_start_main /build/glibc-LK5gWL/glibc-2.23/csu/../csu/libc-start.c:291
    #30 0x4194e8 in _start (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls+0x4194e8)

0x621000071908 is located 0 bytes to the right of 4104-byte region [0x621000070900,0x621000071908)
allocated by thread T0 here:
    #0 0x4b9618 in malloc (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls+0x4b9618)
    #1 0x92c009 in H5MM_malloc /home/fuzzer/victim/hdf5-1.10.5/src/H5MM.c:322:21

SUMMARY: AddressSanitizer: heap-buffer-overflow (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5ls/h5ls+0x4a334c) in __asan_memcpy
Shadow bytes around the buggy address:
  0x0c42800062d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c42800062e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c42800062f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c4280006300: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c4280006310: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c4280006320: 00[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4280006330: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4280006340: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4280006350: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4280006360: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4280006370: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Heap right redzone:      fb
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack partial redzone:   f4
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==5013==ABORTING
fuzzer@thickfuzzer:~/victim/hdf5-1.10.
```

*** h5repack (1.10.5) SEGV in function H5F__close - HDFFV-10926 ***

We found the h5repack (1.10.5 version) SEGV in function H5F__close. I have attached the detailed output of crash and testcase file too.

Compilation steps
```
CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```

** ASAN Output **
```
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$ ./h5repack -V
h5repack: Version 1.10.5

fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$ ./h5repack out/1/crashes/unique/testcase-1570123124-1-16_h5ex_d_compact.h5 /dev/null
ASAN:DEADLYSIGNAL
=================================================================
==26041==ERROR: AddressSanitizer: SEGV on unknown address 0x00000000001c (pc 0x00000078baa1 bp 0x7fff9a76c2b0 sp 0x7fff9a76b540 T0)
    #0 0x78baa0 in H5F__close /home/fuzzer/victim/hdf5-1.10.5/src/H5Fint.c:1939:20
    #1 0x764511 in H5Fclose /home/fuzzer/victim/hdf5-1.10.5/src/H5F.c:674:8
    #2 0x4fb279 in copy_objects /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_copy.c:384:9
    #3 0x4f4ba1 in h5repack /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack.c:54:9
    #4 0x4ed905 in main /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_main.c:771:23
    #5 0x7f07c35b582f in __libc_start_main /build/glibc-LK5gWL/glibc-2.23/csu/../csu/libc-start.c:291
    #6 0x4195c8 in _start (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack+0x4195c8)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /home/fuzzer/victim/hdf5-1.10.5/src/H5Fint.c:1939:20 in H5F__close
==26041==ABORTING
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$
```

*** h5repack (1.10.5) SEGV in function H5T_close_real  - HDFFV-10927 ***

We found the h5repack (1.10.5 version) SEGV in function H5T_close_real. I have attached the detailed output of crash and testcase file too.

Compilation steps
```
CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```
** ASAN Output**
```
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$ ./h5repack -V
h5repack: Version 1.10.5

fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$ ./h5repack out/1/crashes/unique/testcase-1570123276-1-2634_h5ex_d_shuffle.h5 /dev/null
ASAN:DEADLYSIGNAL
=================================================================
==26054==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000008 (pc 0x000000bf9932 bp 0x7ffc4096c910 sp 0x7ffc4096c460 T0)
    #0 0xbf9931 in H5T_close_real /home/fuzzer/victim/hdf5-1.10.5/src/H5T.c:3684:20
    #1 0x1030ed2 in H5O__dset_free_copy_file_udata /home/fuzzer/victim/hdf5-1.10.5/src/H5Doh.c:150:9
    #2 0x98e598 in H5O__copy_header_real /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:928:17
    #3 0x98909a in H5O__copy_header /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:1161:8
    #4 0x98909a in H5O__copy_obj /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:1217
    #5 0x98909a in H5O__copy /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:313
    #6 0x98909a in H5Ocopy /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:232
    #7 0x4fdc03 in do_copy_objects /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_copy.c:1124:25
    #8 0x4fdc03 in copy_objects /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_copy.c:353
    #9 0x4f4ba1 in h5repack /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack.c:54:9
    #10 0x4ed905 in main /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_main.c:771:23
    #11 0x7fd9060b682f in __libc_start_main /build/glibc-LK5gWL/glibc-2.23/csu/../csu/libc-start.c:291
    #12 0x4195c8 in _start (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack+0x4195c8)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /home/fuzzer/victim/hdf5-1.10.5/src/H5T.c:3684:20 in H5T_close_real
==26054==ABORTING
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$
```

*** h5repack (1.10.5) Double Free in function H5MM_xfree - HDFFV-10928 ***

We found the h5repack (1.10.5 version) Double Free in function H5MM_xfree. I have attached the detailed output of crash and testcase file too.

Compilation steps
```
CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```
** ASAN Output**
```
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$ ./h5repack -V
h5repack: Version 1.10.5

fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$ ./h5repack out/2/crashes/unique/testcase-1570123341-2-2935_h5ex_d_checksum.h5 /dev/null
=================================================================
==26121==ERROR: AddressSanitizer: attempting double-free on 0x60c00000bd40 in thread T0:
    #0 0x4b9570 in __interceptor_cfree.localalias.0 (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack+0x4b9570)
    #1 0x952aa7 in H5MM_xfree /home/fuzzer/victim/hdf5-1.10.5/src/H5MM.c:560:9
    #2 0x69ea9e in H5D__chunk_copy /home/fuzzer/victim/hdf5-1.10.5/src/H5Dchunk.c:6089:9
    #3 0x9d4a1e in H5O__layout_copy_file /home/fuzzer/victim/hdf5-1.10.5/src/H5Olayout.c:1059:20
    #4 0x9eb6f7 in H5O__msg_copy_file /home/fuzzer/victim/hdf5-1.10.5/src/H5Omessage.c:1876:29
    #5 0x98f2d6 in H5O__copy_header_real /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:617:44
    #6 0x98909a in H5O__copy_header /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:1161:8
    #7 0x98909a in H5O__copy_obj /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:1217
    #8 0x98909a in H5O__copy /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:313
    #9 0x98909a in H5Ocopy /home/fuzzer/victim/hdf5-1.10.5/src/H5Ocopy.c:232
    #10 0x4fdc03 in do_copy_objects /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_copy.c:1124:25
    #11 0x4fdc03 in copy_objects /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_copy.c:353
    #12 0x4f4ba1 in h5repack /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack.c:54:9
    #13 0x4ed905 in main /home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack_main.c:771:23
    #14 0x7f3c4b8bd82f in __libc_start_main /build/glibc-LK5gWL/glibc-2.23/csu/../csu/libc-start.c:291
    #15 0x4195c8 in _start (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack+0x4195c8)

0x60c00000bd40 is located 0 bytes inside of 128-byte region [0x60c00000bd40,0x60c00000bdc0)
freed by thread T0 here:
    #0 0x4b9a78 in realloc (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack+0x4b9a78)
    #1 0x952629 in H5MM_realloc /home/fuzzer/victim/hdf5-1.10.5/src/H5MM.c:432:21
    #2 0x101908c in H5D__btree_idx_iterate_cb /home/fuzzer/victim/hdf5-1.10.5/src/H5Dbtree.c:1095:21
    #3 0x1008fd5 in H5B__iterate_helper /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1165:25
    #4 0x1008a22 in H5B_iterate /home/fuzzer/victim/hdf5-1.10.5/src/H5B.c:1210:21
    #5 0x69df90 in H5D__chunk_copy /home/fuzzer/victim/hdf5-1.10.5/src/H5Dchunk.c:6050:8
    #6 0x9d4a1e in H5O__layout_copy_file /home/fuzzer/victim/hdf5-1.10.5/src/H5Olayout.c:1059:20

previously allocated by thread T0 here:
    #0 0x4b96f8 in malloc (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack+0x4b96f8)
    #1 0x952529 in H5MM_malloc /home/fuzzer/victim/hdf5-1.10.5/src/H5MM.c:322:21
    #2 0x9d4a1e in H5O__layout_copy_file /home/fuzzer/victim/hdf5-1.10.5/src/H5Olayout.c:1059:20

SUMMARY: AddressSanitizer: double-free (/home/fuzzer/victim/hdf5-1.10.5/tools/src/h5repack/h5repack+0x4b9570) in __interceptor_cfree.localalias.0
==26121==ABORTING
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5repack$
```

All above issues are reported on 3/4-Oct-2019.
