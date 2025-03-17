The haystack is just a polynomial: product of (x + p) over p random primes.
The RSA factors are the only (not necessarily, but quite surely) repeated roots of the polynomial.
The gcd of the polynomial with its derivative will be (x + p) * (x + q), where p and q are the factors.
This allows us to recover both the modulus and the private exponent.
Alternatively, one could find all the roots and try their pairwise combinations to decrypt.
That may be feasible, but - having the "accidentally" repeated roots - it's not necessary.
Care must be taken in computing the gcd: floats don't have enough range, and fractions may be too slow.

```Python
from base64 import b64decode
from gmpy2 import gcd
from math import isqrt

b64dec = lambda y: int(b64decode(y).hex(), 16)

c = input("ciphertext>").split()[-1]
c = int(c, 16)

size = input("size>").split()[-1]
size = int(size)

r = []
for _ in range(size):
	r.append(b64dec(input()))

print("\ncomputing:")

print("derivative...")
s = []
for k, x in enumerate(r, 1):
	deg = size - k
	s.append(deg * x)
if s:
	assert not s[-1]
	s.pop()

def poly_gcd(a, b):
	if len(a) < len(b):
		a, b = b, a
	counter = 0
	while b:
		assert a[0]
		assert b[0]
		if len(a) < len(b):
			a, b = b, a
			counter -= 1
		counter += 1
		if counter > 1:
			d = gcd(*a)
			for i in range(len(a)):
				a[i] //= d
			d = gcd(*b)
			for i in range(len(b)):
				b[i] //= d
			counter = 0
		d = gcd(a[0], b[0])
		qa = b[0] // d
		qb = a[0] // d
		a[0] = 0
		for i in range(1, len(b)):
			a[i] = (qa * a[i]) - (qb * b[i])
		for i in range(len(b), len(a)):
			a[i] *= qa
		cleared = 0
		for x in a:
			if x:
				break
			else:
				cleared += 1
		a, b = b, a[cleared:]
	return a

print("gcd...")
a, b, n = poly_gcd(r, s)
#print("---", a)
#print("---", b)
#print("---", n)
b, rb = divmod(b, a)
assert not rb
n, rn = divmod(n, a)
assert not rn

print("modulus...")
print(n)

print("factors...")
avg, r = divmod(-b, 2)
assert not r
hd = isqrt(avg**2 - n)
x1 = avg - hd
x0 = avg + hd
p, q = -x0, -x1
print(p)
print(q)
assert p > 0
assert q > 0
assert n == p * q

print("private exponent...")
e = pow(65537, -1, (p - 1) * (q - 1))
print(e)

print("message...")
m = pow(c, e, n)
h = hex(m)[2:]
if len(h) % 2:
	h = '0' + h
m = bytes.fromhex(h)
print(m)
print(m.decode())
```
