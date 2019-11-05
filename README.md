The purpose of this post is to explain the answer to the question
"Define a lambda expression that raises an Exception"
https://stackoverflow.com/questions/8294618/define-a-lambda-expression-that-raises-an-exception

# The answer in question
Answer by Marcelo Cantos
```python
type(lambda: 0)(type((lambda: 0).__code__)(
    1,0,1,1,67,b'|\0\202\1\0',(),(),('x',),'','',1,b''),{}
)(Exception())
```

# Explanation
`lambda: 0` is an instance of the `builtins.function` class.  
`type(lambda: 0)` is the `builtins.function` class.  
`(lambda: 0).__code__` is a `code` object.  
A `code` object is an object which holds the compiled bytecode among other things.
It is defined here in CPython https://github.com/python/cpython/blob/master/Include/code.h.
Its methods are implemented here https://github.com/python/cpython/blob/master/Objects/codeobject.c.
We can run the help on the code object:
```
Help on code object:

class code(object)
 |  code(argcount, kwonlyargcount, nlocals, stacksize, flags, codestring,
 |        constants, names, varnames, filename, name, firstlineno,
 |        lnotab[, freevars[, cellvars]])
 |  
 |  Create a code object.  Not for the faint of heart.
```
`type((lambda: 0).__code__)` is the code class.  
So when we say
```python
type((lambda: 0).__code__)(
    1,0,1,1,67,b'|\0\202\1\0',(),(),('x',),'','',1,b'')
```
we are calling the constructor of the code object with the following arguments:
* argcount=1
* kwonlyargcount=0
* nlocals=1
* stacksize=1
* flags=67
* codestring=b'|\0\202\1\0'
* constants=()
* names=()
* varnames=('x',)
* filename=''
* name=''
* firstlineno=1
* lnotab=b''

You can read about what the arguments mean in the definition of the `PyCodeObject`
https://github.com/python/cpython/blob/master/Include/code.h.
The value of 67 for the `flags` argument is for example `CO_OPTIMIZED | CO_NEWLOCALS | CO_NOFREE`.

The most importand argument is the `codestring` which contains instruction opcodes.
Let's see what they mean.
```python
>>> import dis
>>> dis.dis(b'|\0\202\1\0')
          0 LOAD_FAST                0 (0)
          2 RAISE_VARARGS            1
          4 <0>
```
The documentation of opcodes can by found here
https://docs.python.org/3.8/library/dis.html#python-bytecode-instructions.
The first byte is the opcode for `LOAD_FAST`, the second byte is its argument i.e. 0.
```
LOAD_FAST(var_num)
    Pushes a reference to the local co_varnames[var_num] onto the stack.
```

So we push the reference to 'x' onto the stack. The `varnames` is a list of strings containing only 'x'.
We will push the only argument of the function we are defining to the stack.

The next byte is the opcode for `RAISE_VARAGS` and the next byte is its argument i.e. 1.
```
RAISE_VARARGS(argc)
    Raises an exception using one of the 3 forms of the raise statement, depending on the value of argc:
        0: raise (re-raise previous exception)
        1: raise TOS (raise exception instance or type at TOS)
        2: raise TOS1 from TOS (raise exception instance or type at TOS1 with __cause__ set to TOS)
```

The TOS is the top-of-stack.
Since we pushed the first argument (`x`) of our function to the stack and `argc` is 1 we will raise the
`x` if it is an exception instance or make an instance of `x` and raise it otherwise.

The last byte i.e. 0 is not used. It is not a valid opcode. It might as well not be there.

Going back to code snippet we are anylyzing:
```python
type(lambda: 0)(type((lambda: 0).__code__)(
    1,0,1,1,67,b'|\0\202\1\0',(),(),('x',),'','',1,b''),{}
)(Exception())
```
We called the constructor of the code object:
```python
type((lambda: 0).__code__)(
    1,0,1,1,67,b'|\0\202\1\0',(),(),('x',),'','',1,b'')
```
We pass the code object and an empty dictionary to the constructor of a function object:
```python
type(lambda: 0)(type((lambda: 0).__code__)(
    1,0,1,1,67,b'|\0\202\1\0',(),(),('x',),'','',1,b''),{}
)
```
Let's call help on a function object to see what the arguments mean.
```
Help on class function in module builtins:

class function(object)
 |  function(code, globals, name=None, argdefs=None, closure=None)
 |  
 |  Create a function object.
 |  
 |  code
 |    a code object
 |  globals
 |    the globals dictionary
 |  name
 |    a string that overrides the name from the code object
 |  argdefs
 |    a tuple that specifies the default argument values
 |  closure
 |    a tuple that supplies the bindings for free variables
```

We then call the constructed function passing an Exception instance as an argument.
Consequently we called a lambda function which raises an exception.
Let's run the snippet and see that it indeed works as intended.
```python
>>> type(lambda: 0)(type((lambda: 0).__code__)(
...     1,0,1,1,67,b'|\0\202\1\0',(),(),('x',),'','',1,b''),{}
... )(Exception())
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
  File "", line 1, in 
Exception
```
# Improvements
We saw that the last byte of the bytecode is useless. Let's not clutter this
complicated expression needlesly. Let's remove that byte.
Also if we want to golf a little we could omit the instantiation of Exception
and instead pass the Exception class as an argument. Those changes would result
in the following code:
```python
type(lambda: 0)(type((lambda: 0).__code__)(
    1,0,1,1,67,b'|\0\202\1',(),(),('x',),'','',1,b''),{}
)(Exception)
```
When we run it we will get the same result as before. It's just cleaner.

Author: Michał Nienański
# Bibliography
* https://stackoverflow.com/questions/8294618/define-a-lambda-expression-that-raises-an-exception
* https://github.com/python/cpython/blob/master/Objects/codeobject.c
* https://github.com/python/cpython/blob/master/Include/code.h
