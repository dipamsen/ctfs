# Jail Break

_We've built the world's most secure Python sandbox. Nothing can escape. Probably. Hopefully. Run it locally: python3 jail.py_

_By Garv (@GarvK07 on discord)_

Attachments: `jail.py`

---

```py
import sys

_ENC = [0x37, 0x36, 0x24, 0x2e, 0x23, 0x25, 0x39, 0x32, 0x3b, 0x1d, 0x28, 0x23, 0x73, 0x2e, 0x1d, 0x71, 0x31, 0x21, 0x76, 0x32, 0x71, 0x1d, 0x2f, 0x76, 0x31, 0x36, 0x71, 0x30, 0x3f]
_KEY = 0x42

def _secret():
    return ''.join(chr(b ^ _KEY) for b in _ENC)

BANNED = [
    "import", "os", "sys", "system", "eval",
    "open", "read", "write", "subprocess", "pty",
    "popen", "secret", "_enc", "_key"
]

SAFE_BUILTINS = {
    "print": print,
    "input": input,
    "len": len,
    "str": str,
    "int": int,
    "chr": chr,
    "ord": ord,
    "range": range,
    "type": type,
    "dir": dir,
    "vars": vars,
    "getattr": getattr,
    "setattr": setattr,
    "hasattr": hasattr,
    "isinstance": isinstance,
    "enumerate": enumerate,
    "zip": zip,
    "map": map,
    "filter": filter,
    "list": list,
    "dict": dict,
    "tuple": tuple,
    "set": set,
    "bool": bool,
    "bytes": bytes,
    "hex": hex,
    "oct": oct,
    "bin": bin,
    "abs": abs,
    "min": min,
    "max": max,
    "sum": sum,
    "sorted": sorted,
    "reversed": reversed,
    "repr": repr,
    "hash": hash,
    "id": id,
    "callable": callable,
    "iter": iter,
    "next": next,
    "object": object,
}

# _secret is in globals but not documented - players must find it
GLOBALS = {"__builtins__": SAFE_BUILTINS, "_secret": _secret}

print("=" * 50)
print("  Welcome to PyJail v1.0")
print("  Escape to get the flag!")
print("=" * 50)
print()

while True:
    try:
        code = input(">>> ")
    except EOFError:
        break

    blocked = False
    for word in BANNED:
        if word.lower() in code.lower():
            print(f"  [BLOCKED] Nice try!")
            blocked = True
            break

    if blocked:
        continue

    try:
        exec(compile(code, "<jail>", "exec"), GLOBALS)
    except Exception as e:
        print(f"  [ERROR] {e}")
```

The script provides a repl where the user cannot write certain banned words. To get the secret, the user must call the `_secret` function (which is accessible in the evaluation scope).

Since the word `secret` is banned, we have to construct the string `_secret` ourselves. But, we can't use `eval` or similar (those are banned too) to evaluate the string as a variable name.

Instead we will use python builtins. `vars()` is useful here. Calling it without any arguments returns a dictionary of all the variables in the current scope.

```
>>> print(vars())
{'__builtins__': {'print': <built-in function print>, 'input': <built-in function input>, 'len': <built-in function len>, 'str': <class 'str'>, 'int': <class 'int'>, 'chr': <built-in function chr>, 'ord': <built-in function ord>, 'range': <class 'range'>, 'type': <class 'type'>, 'dir': <built-in function dir>, 'vars': <built-in function vars>, 'getattr': <built-in function getattr>, 'setattr': <built-in function setattr>, 'hasattr': <built-in function hasattr>, 'isinstance': <built-in function isinstance>, 'enumerate': <class 'enumerate'>, 'zip': <class 'zip'>, 'map': <class 'map'>, 'filter': <class 'filter'>, 'list': <class 'list'>, 'dict': <class 'dict'>, 'tuple': <class 'tuple'>, 'set': <class 'set'>, 'bool': <class 'bool'>, 'bytes': <class 'bytes'>, 'hex': <built-in function hex>, 'oct': <built-in function oct>, 'bin': <built-in function bin>, 'abs': <built-in function abs>, 'min': <built-in function min>, 'max': <built-in function max>, 'sum': <built-in function sum>, 'sorted': <built-in function sorted>, 'reversed': <class 'reversed'>, 'repr': <built-in function repr>, 'hash': <built-in function hash>, 'id': <built-in function id>, 'callable': <built-in function callable>, 'iter': <built-in function iter>, 'next': <built-in function next>, 'object': <class 'object'>}, '_secret': <function _secret at 0x7b52fd21ce00>}
```

We can see the `_secret` function in sight! So we can simply call it:

```
>>> print(vars()['_' + 's' + 'ecret']())
utflag{py_ja1l_3sc4p3_m4st3r}
```

## another solution

later i realised, you can just edit the `jail.py` file and add a `print(_secret())` (for purely getting the flag) 🥀