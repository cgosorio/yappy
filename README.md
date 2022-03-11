# yappy

Yet Another Parser Generator for Python.

Yappy provides a lexical analyser and a LR parser generator
for Python applications. It currently builds SLR,
LR(1) and LALR(1) tables. Tables are kept in Python shelves for use
in parsing.  Some ambiguous grammars can be handled if priority
and associativity information is provided.

_The original buggy version_ can be installed on a linux box with:
```bash
sudo apt install python-yappy
```

And the documentation with:
```bash
sudo apt install python-yappy-doc
```

To use this corrected version, clone this repository.

Yappy is a project written by [Nelma Moreira](http://www.dcc.fc.up.pt/~nam/)
and [Rogério Reis](http://www.dcc.fc.up.pt/~rvr/). More information about
the parser can be found in the paper [Yappy Yet another LR(1) parser generator for Python](https://www.researchgate.net/publication/237445856_Yappy_Yet_another_LR1_parser_generator_for_Python).

The available implementation had some errors and although I have tried to inform 
the authors about them, I have not get any answer yet, and the errors seem
to be still present in the sources.

This repository is just to
post the corrections so other yappy users can benefit from them.

Also, **now the code works with Python3!**.

## Error 1: wrong indentation

This is the portion of code with the error:
```python
 486                      for k in range(len(r)-1): 
 487                          j = k + 1 
 488                          if r[k] in self.nonterminals and self.nullable[string.join(r[j:])]: 
 489                              if self.follow[r[k]].s_extend(self.follow[s]): 
 490                                  e = 1 
 491                              break 
```

The problem is the break at line 491, it should be indented an extra tab,
within the if. The corrected code would be:
```python
 486                      for k in range(len(r)-1): 
 487                          j = k + 1 
 488                          if r[k] in self.nonterminals and self.nullable[string.join(r[j:])]: 
 489                              if self.follow[r[k]].s_extend(self.follow[s]): 
 490                                  e = 1 
 491                                  break 
```

For example with this grammar:
```
S --> B C D A 
A --> n A | ε 
B --> t 
C --> b D e | ε 
D --> i E | ε 
E --> S f | p 
```

Yappy calculates as FOLLOW(C) the set: {i, n}, but the correct
FOLLOW(C) is {i, n, $, f}

In this grammar the nullable symbols are A, C, and D, so from first
production in FOLLOW(C) Yappy should have also included the FOLLOW(S)={$,f} since
both D and A are nullable, and those two symbols are the ones missing in the
original implementation.


## Error 2: Wrong condition

In the constructor for Yappy there is another error. The following condition
is not well constructed:
```python
if (self.Log.noconflicts and
        ((self.Log.conflicts.has_key('sr') and
            len(self.Log.conflicts['sr'])!=self.Log.expect) or
            self.Log.conflicts.has_key('rr'))):
    print "LR conflicts: number %s value %s" %(len(self.Log.conflicts['sr']),self.Log.conflicts)
    print """If it is Ok, set expect to the number of conflicts and build table again"""
```
The condition can be True having a Reduce-Reduce conflict, what makes True the
test `self.Log.conflicts.has_key('rr')`, that makes all the condition True,
even if there is not `self.Log.conflicts['sr']` (that is, even if there is not
Shift-Reduce conflicts). Then when the print is executed a `keyerror` exception
is thrown.

The exception can be launched using this grammar:
```python
Yappy([],"A -> B C; B -> ; B -> A b; C -> ; C -> c; A -> a;")
```

The corrected code is now:
```python
if self.Log.noconflicts:
    n_sr = len(self.Log.conflicts.get('sr', []))
    n_rr = len(self.Log.conflicts.get('rr', []))
    if n_sr + n_rr > self.Log.expect:
        print("LR conflicts: number %s value %s" %
              (n_sr+n_rr,self.Log.conflicts))
        print("If it is Ok, set expect to the number of",
              "conflicts and build table again")
```
