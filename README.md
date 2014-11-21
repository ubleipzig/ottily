ottily
======

Ottily executes a javascript snippet on each line of an input file in parallel.

Examples
--------

Noop -- is just an expensive `cat`.

    $ ottily datasets/simple.ldj
    {"name": "ottily", "language": "Golang"}

Inline script with `-e`:

    $ ottily -e 'output=input.length' datasets/simple.ldj
    40

    $ ottily -e 'o=JSON.parse(input); o["language"] = "Go"; output=JSON.stringify(o);' \
                datasets/simple.ldj

    {"language":"Go","name":"ottily"}

Pass a script file with `-s`:

    $ cat scripts/classified.js
    output = "CLASSIFIED"

    $ ottily -s scripts/classified.js datasets/simple.ldj
    CLASSIFIED

A string variable `input` is passed into the javascript snippet.
To produce output, set the `output` variable to the desired string value.
If `output` is set to null or is undefined, nothing is printed. That's it.

By default, `ottily` will run as many workers as there are cores.

Performance
-----------

Ottily is just a [156 LOC](https://github.com/miku/ottily/blob/6d81c71afe2a29fb5d3445b0813642285463ca6b/cmd/ottily/main.go) Go program. In many cases, there will be faster alternatives.

Given a file with 1 million lines, calculate the length of each line.

    $ time perl -ne 'print length . "\n";' datasets/1M.ldj > /dev/null
    real    0m2.085s
    user    0m1.588s
    sys     0m0.495s

    $ time awk '{ print length }' datasets/1M.ldj  > /dev/null

    real    0m5.836s
    user    0m5.514s
    sys     0m0.314s

    $ time ottily -e 'output=input.length' datasets/1M.ldj > /dev/null

    real    0m16.488s
    user    0m53.391s
    sys     0m7.171s

Given a file with 1 million lines, one JSON document per line, add a new key to each JSON document.

    $ time jq -c '."about" = "jq"' datasets/1M.ldj > /dev/null

    real    2m31.063s
    user    2m29.517s
    sys     0m1.500s

    $ time ottily -e 'o=JSON.parse(input); o["about"] = "ot"; output=JSON.stringify(o);' \
                  datasets/1M.ldj > /dev/null

    real    5m59.872s
    user    21m3.009s
    sys     0m52.241s

Credits
-------

Ottily uses [otto](https://github.com/robertkrimen/otto) to run js.
