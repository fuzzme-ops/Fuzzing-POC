## Heap Buffer Overflow in h5stat 1.10.5 - HDFFV-10921 
We found the Heap Buffer Overflow in h5stat 1.10.5 version. I have attached the detailed output of crash and testcase file too.

**Compilation steps**
```
CC=afl-clang-fast CXX=afl-clang-fast++ ASAN_OPTIONS=symbolize=1:detect_leaks=1 ./configure
AFL_USE_ASAN=1 ASAN_SYMBOLIZER_PATH=/usr/lib/llvm-3.8/bin/llvm-symbolizer make
Test System:  Ubuntu 16 (4.4.0-87-generic)
```

**ASAN Output**
```
fuzzer@thickfuzzer:~/victim/hdf5-1.10.5/tools/src/h5stat$ ./h5stat -V
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
