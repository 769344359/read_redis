#!/bin/sh
awk '{array[NR]=$4 }END {for (i in array) print array[NR-i +1]} '  Untitled-1.txt
