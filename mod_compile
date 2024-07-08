#!/bin/bash

if [ "$#" -eq 0 ] || [ "$1" = "--help" ]
then
	echo "mod_compile <xyz>.clsp"
	exit 1
fi

CLSP=$(basename $1)
DIR=$(dirname $1)
OUTDIR=$DIR/output

if [ ! -f "$DIR/$CLSP" ]
then
	echo $DIR/$CLSP: file not found
	exit 1
fi

if [ ! -d "$OUTDIR" ]
then 
	mkdir "$OUTDIR"
fi

if [ "$DIR/$CLSP" -nt "$OUTDIR/$CLSP.clvm" ]
then
	~/.cargo/bin/run -O -i "$DIR/include/" -i "$OUTDIR/" "$DIR/$CLSP" > "$OUTDIR/$CLSP.clvm"
	if [ "$(head -c 4 "$OUTDIR/$CLSP.clvm")" = "FAIL" ]
	then
		echo "Compilation error:"
		cat "$OUTDIR/$CLSP.clvm"
		exit 1
	fi
fi

if [ "$OUTDIR/$CLSP.clvm" -nt "$OUTDIR/$CLSP.hex" ]
then
	~/.cargo/bin/opc "$OUTDIR/$CLSP.clvm" > "$OUTDIR/$CLSP.hex"
fi

if [ "$OUTDIR/$CLSP.clvm" -nt "$OUTDIR/$CLSP.hash" ]
then
	~/.cargo/bin/opc -H "$OUTDIR/$CLSP.clvm" > "$OUTDIR/$CLSP.hash"
fi

if [ "$OUTDIR/$CLSP.hash" -nt "$OUTDIR/$CLSP.hash.bin" ]
then
	xxd -r -p "$OUTDIR/$CLSP.hash" "$OUTDIR/$CLSP.hash.bin"
fi

echo Success!
