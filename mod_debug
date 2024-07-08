#!/bin/bash

if [ "$#" -lt 2 ] || [ "$1" = "--help" ]
then 
	echo "mod_debug <xyz>.clsp testname"
	echo ""
	echo "Files in ./tests/ for each test:"
	echo "  <xyz>.clsp.<testname>.curry (optional)"
	echo "  <xyz>.clsp.<testname>.env (environment/solution)"
	echo "  <xyz>.clsp.<testname>.expected (expected output)"
	echo "Each of these files can be generated with:"
	echo "  <xyz>.clsp.<testname>.gen_cur.clsp"
	echo "  <xyz>.clsp.<testname>.gen_env.clsp"
	echo "  <xyz>.clsp.<testname>.gen_exp.clsp"
	echo ""
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

if [ ! -f "$OUTDIR/$CLSP.clvm" ]
then
        echo $OUTDIR/$CLSP.clvm: file not found
        echo Run mod_compile to generate this file
        exit 1
fi

if [ "$DIR/$CLSP" -nt "$OUTDIR/$CLSP.clvm" ]
then
	echo $OUTDIR/$CLSP.clvm: out of date
	echo Run mod_compile to regenerate this file
	exit 1
fi

TEST=$2

if [ ! -d "$OUTDIR" ]; then mkdir "$OUTDIR"; fi

CLDB_TARGET=""
if [ -f "$DIR/tests/$CLSP/$TEST.curry" ]
then
	~/.cargo/bin/curry "$(cat $OUTDIR/$CLSP.clvm)" "$(cat $DIR/tests/$CLSP/$TEST.curry)" > "$OUTDIR/$CLSP.$TEST.curried"
	~/.cargo/bin/opc "$OUTDIR/$CLSP.$TEST.curried" > "$OUTDIR/$CLSP.$TEST.curried.hex"
	CLDB_TARGET="$OUTDIR/$CLSP.$TEST.curried.hex"
elif [ -f "$DIR/tests/$CLSP/$TEST.gen_cur.clsp" ]
then
	~/.cargo/bin/run "$DIR/tests/$CLSP/$TEST.gen_cur.clsp" > "$OUTDIR/$CLSP.$TEST.gen_cur.clsp.clvm"
	if [ "$(head -c 4 "$OUTDIR/$CLSP.$TEST.gen_cur.clsp.clvm")" = "FAIL" ]
	then
		echo "Compilation error on curry generator:"
		cat "$OUTDIR/$CLSP.$TEST.gen_cur.clsp.clvm"
		exit 1
	fi
	~/.cargo/bin/opc "$OUTDIR/$CLSP.$TEST.gen_cur.clsp.clvm" > "$OUTDIR/$CLSP.$TEST.gen_cur.clsp.hex"
	~/.cargo/bin/brun "$OUTDIR/$CLSP.$TEST.gen_cur.clsp.clvm" > "$OUTDIR/$CLSP.$TEST.curry"
	~/.cargo/bin/curry "$(cat $OUTDIR/$CLSP.clvm)" "$(cat $OUTDIR/$CLSP.$TEST.curry)" > "$OUTDIR/$CLSP.$TEST.curried"
	~/.cargo/bin/opc "$OUTDIR/$CLSP.$TEST.curried" > "$OUTDIR/$CLSP.$TEST.curried.hex"
	CLDB_TARGET="$OUTDIR/$CLSP.$TEST.curried.hex"
else
	CLDB_TARGET="$OUTDIR/$CLSP.hex"
fi

echo "()" > "$OUTDIR/no_env.env"
~/.cargo/bin/opc "$OUTDIR/no_env.env" > "$OUTDIR/no_env.env.hex"

CLDB_ENV=
if [ -f "$DIR/tests/$CLSP/$TEST.env" ]
then
	~/.cargo/bin/opc "$DIR/tests/$CLSP/$TEST.env" > "$OUTDIR/$CLSP.$TEST.env.hex"
	CLDB_ENV="$OUTDIR/$CLSP.$TEST.env.hex"
elif [ -f "$DIR/tests/$CLSP/$TEST.gen_env.clsp" ]
then
	~/.cargo/bin/run "$DIR/tests/$CLSP/$TEST.gen_env.clsp" > "$OUTDIR/$CLSP.$TEST.gen_env.clsp.clvm"
	if [ "$(head -c 4 "$OUTDIR/$CLSP.$TEST.gen_env.clsp.clvm")" = "FAIL" ]
	then
		echo "Compilation error on environment generator:"
		cat "$OUTDIR/$CLSP.$TEST.gen_env.clsp.clvm"
		exit 1
	fi
	~/.cargo/bin/opc "$OUTDIR/$CLSP.$TEST.gen_env.clsp.clvm" > "$OUTDIR/$CLSP.$TEST.gen_env.clsp.hex"
	~/.cargo/bin/brun "$OUTDIR/$CLSP.$TEST.gen_env.clsp.clvm" > "$OUTDIR/$CLSP.$TEST.env"
	~/.cargo/bin/opc "$OUTDIR/$CLSP.$TEST.env" > "$OUTDIR/$CLSP.$TEST.env.hex"
	CLDB_ENV="$OUTDIR/$CLSP.$TEST.env.hex"
else
	CLDB_ENV="$OUTDIR/no_env.env.hex"
fi


EXP_FILE=
if [ -f "$DIR/tests/$CLSP/$TEST.expected" ]
then
	EXP_FILE="$DIR/tests/$CLSP/$TEST.expected"
elif [ -f "$DIR/tests/$CLSP/$TEST.gen_exp.clsp" ]
then
	~/.cargo/bin/run "$DIR/tests/$CLSP/$TEST.gen_exp.clsp" > "$OUTDIR/$CLSP.$TEST.gen_exp.clsp.clvm"
	if [ "$(head -c 4 "$OUTDIR/$CLSP.$TEST.gen_exp.clsp.clvm")" = "FAIL" ]
	then
		echo "Compliation error on expected file generator:"
		cat "$OUTDIR/$CLSP.$TEST.gen_exp.clsp.clvm"
		exit 1
	fi
	~/.cargo/bin/opc "$OUTDIR/$CLSP.$TEST.gen_exp.clsp.clvm" > "$OUTDIR/$CLSP.$TEST.gen_exp.clsp.hex"
	~/.cargo/bin/brun "$OUTDIR/$CLSP.$TEST.gen_exp.clsp.clvm" > "$OUTDIR/$CLSP.$TEST.expected"
	EXP_FILE="$OUTDIR/$CLSP.$TEST.expected"
else
	echo "Error: missing .expected file"
	exit 1
fi

echo Expected Output:
cat "$EXP_FILE"
echo ""
echo Debugger Output:
echo ~/.cargo/bin/cldb -x -p "$CLDB_TARGET" "$CLDB_ENV"
~/.cargo/bin/cldb -x -p "$CLDB_TARGET" "$CLDB_ENV" | tee "$OUTDIR/$CLSP.$TEST.out"


