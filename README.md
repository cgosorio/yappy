# yappy
Yet Another Parser Generator for Python.

Yappy provides a lexical analyser and a LR parser generator
for Python applications. It currently builds SLR,
LR(1) and LALR(1) tables. Tables are kept in Python shelves for use
in parsing.  Some ambiguous grammars can be handled if priority
and associativity information is provided.

It can be installed on a linux box with:
```bash
sudo apt install python-yappy
```

And the documentation with:
```bash
sudo apt install python-yappy-doc
```

This version has some small corrections, and who knows eventually I will find
the time to change the code to make it compatible with Python 3.

