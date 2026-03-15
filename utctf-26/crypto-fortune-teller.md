# Fortune Teller

_Our security team built a "cryptographically secure" random number generator. The lead engineer assured us it was basically AES. He has since been let go._

_By Garv (@GarvK07 on discord)_

Attachments: `lcg.txt`

---

```txt
We're using a Linear Congruential Generator (LCG) defined as:

  x_(n+1) = (a * x_n + c) % m

where m = 4294967296 (2^32), and a and c are secret.

We intercepted the first 4 outputs of the generator:

  output_1 = 4176616824
  output_2 = 2681459949
  output_3 = 1541137174
  output_4 = 3272915523

The flag was encrypted by XORing it with output_5 (used as a 4-byte repeating key).

  ciphertext (hex) = 3cff226828ec3f743bb820352aff1b7021b81b623cff31767ad428672ef6
```

We have

$$
x_2 = (a x_1  + c) \mod{m}$$$$
x_3 = (a x_2  + c) \mod{m}$$$$
x_4 = (a x_3  + c) \mod{m}$$$$
$$

So 

$$
x_3 - x_2 = a (x_2 - x_1) \mod{m}$$$$
\Rightarrow a = (x_2 - x_1)^{-1} (x_3 - x_2) \mod{m}
$$

```
>>> a = pow(x2 - x1, -1, m) * (x3 - x2) % m
>>> a
3355924837
```

and

$$
c = x_2 - a x_1 \mod{m}
$$

```
>>> c = (x2 - a * x1) % m
>>> c
2915531925
```

Now we can find $x_5$:

$$
x_5 = a x_4 + c \mod{m}
$$

```
>>> x5 = (a * x4 + c) % m
>>> x5
1233863684
```

now for the fun part

```
>>> b = x5.to_bytes(4, "big")
>>> c = bytes.fromhex("3cff226828ec3f743bb820352aff1b7021b81b623cff31767ad428672ef6")
>>> bytes(b[i % 4] ^ c[i] for i in range(len(c)))
b'utflag{pr3d1ct_th3_futur3_lcg}'
```