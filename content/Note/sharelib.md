---
title: "2018-01-25 where share library are searched"
date: 2018-01-25 17:03
---

most of the words are extract from the document `Executable and Linkable Format`

###how dependency is recorded 

When the dynamic linker creates the memory segments for an object file, the dependencies (recorded in DT_NEEDED entries of the dynamic structure) tell what shared objects are needed to supply the program’s services.

###where share object file is searched
If a shared object name has one or more slash (/) characters anywhere in the name, such as /usr/lib/lib2 above or directory/file, the dynamic linker uses that string directly as the path name. If the name has no slashes, are searched in the share library search path. may be config in `/etc/ld.so.conf`.

###see a elf's share object dependency

```
ldd file
```