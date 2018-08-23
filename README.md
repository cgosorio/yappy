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

## The wrong indentation error

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


## The wrong condition error

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
        print "LR conflicts: number %s value %s" %(n_sr+n_rr,self.Log.conflicts)
        print """If it is Ok, set expect to the number of conflicts and build table again"""
```
