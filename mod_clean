#!/bin/bash

if [ "$#" -eq 0 ] || [ "$1" = "--help" ]
then
	echo "mod_clean <xyz>.clsp"
	echo "to clean all output files for one clsp file"
	echo "  or"
	echo "mod_clean <dir_with_clsp_files>"
	echo "to clean output files for all clsp files in that dir"
	exit 1
fi

if [ -d "$1" ]
then
	rm $(realpath $1)/output/* 2> /dev/null
	rm $(realpath $1)/main.sym 2> /dev/null
	echo "$1/output: cleaned"
	exit 0
fi

CLSP=$(basename $1)
DIR=$(dirname $1)
OUTDIR=$DIR/output

if [ -f "$OUTDIR/$CLSP.clvm" ]
then
	rm $DIR/output/$CLSP.* 2> /dev/null 
	rm $DIR/main.sym 2> /dev/null
        echo "$DIR/$CLSP: cleaned"
        exit 0
fi

echo Nothing found to clean
exit 1
