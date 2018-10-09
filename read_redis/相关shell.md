#!/bin/sh
awk '{array[NR]=$4 }END {for (i in array) print array[NR-i]} '  Untitled-1.txt
