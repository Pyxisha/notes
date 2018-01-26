---
title: "2018-01-17 effictive user and real user"
date: 2018-01-17 21:56
---


when we execute a program file, the effective user id of the process is usually the real user ID,
and the effective group ID is usually the real group id. However we can also set a special flag in 
the file's mode word(st_mode) that says, when this file is executed, set the effective user id of the 
process to be the owner of the file(st_uid). similarly, we can set another bit in the file's mode that 
causes the effective group id to be group owner of the file(st_gid).
