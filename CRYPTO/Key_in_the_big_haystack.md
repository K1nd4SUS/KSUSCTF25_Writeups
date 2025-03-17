Same as before (gcd between the polynomial and its derivative), but a simple Euclidean algorithm is too slow.
A solution could have been binary search for the zeros.
But this time non-double solution are pairwise at distance 2 so a negative value is difficult to find.
Increasing / decreasing could be used instead of positive / negative.
That just means binary search on the derivative: we are interested to solutions that are an odd integer.
Newton's method may also be a usefool tool (do integer divisions; floats are bad here).
The attached solver uses a different approach that makes the original idea competitive again.
We can compute the same polynomial gcd, but over finite fields Z/pZ, for some primes p.
The primes should be big, so the product is over `2**1024` - the infinity norm of (x + p) * (x + q) - with only a few primes (and few gcd computations).
The primes should be small, so that the operations with big nums are not expensive.
A good compromise is to take the 19 biggest primes less than `2**63` (taking only 17 is enough).
Then the second-degree polynomial can be recovered by Chinese Reminder Theorem, using the extended Euclidean algorithm for integers.


```Python
from base64 import b64decode
from gmpy2 import is_prime, gcdext
from math import isqrt

b64dec = lambda y: int(b64decode(y).hex(), 16)
primes = [(1<<63) - k for k in range(1, 800, 2)]
primes = [x for x in primes if is_prime(x)]
prime_name = lambda p: f"P{(1<<63) - p:03d}"

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

r = tuple(r)
s = tuple(s)

def poly_gcd(a, b, p):
	# reduce a
	a = [x % p for x in a]
	cleared = 0
	for x in a:
		if x:
			break
		else:
			cleared += 1
	a = a[cleared:]
	# reduce b
	b = [x % p for x in b]
	cleared = 0
	for x in b:
		if x:
			break
		else:
			cleared += 1
	b = b[cleared:]
	# main loop
	if len(a) < len(b):
		a, b = b, a
	while b:
		assert a[0]
		assert b[0]
		if len(a) < len(b):
			a, b = b, a
		q = a[0] * pow(b[0], -1, p)
		q %= p
		a[0] = 0
		for i in range(1, len(b)):
			x = a[i]
			y = (q * b[i]) % p
			a[i] = x - y if x >= y else x + p - y
		cleared = 0
		for x in a:
			if x:
				break
			else:
				cleared += 1
		a, b = b, a[cleared:]
	return a

print("gcd...")
records = []
for p in primes:
	print(f"\t{prime_name(p)}: ", end="", flush=True)
	ans = poly_gcd(r, s, p)
	n_ans = len(ans) - 1
	print(n_ans)
	if n_ans != 2:
		continue
	a, b, n = poly_gcd(r, s, p)
	ai = pow(a, -1, p)
	b = (b * ai) % p
	n = (n * ai) % p
#	print("---", f"{b = }")
#	print("---", f"{n = }")
	records.append((p, b, n))

print("recompose...")
b, n = 0, 0
mod = 1
for p, bp, np in records:
	one, cp, cmod = map(int, gcdext(p, mod))
	assert one == 1
	assert one == cp * p + cmod * mod
	n *= cp * p
	n += cmod * mod * np
	b *= cp * p
	b += cmod * mod * bp
	mod *= p
	n %= mod
	b %= mod
#	print("---", f"{b = }")
#	print("---", f"{n = }")
#	print("---", f"{mod = }")
#	print("---")

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
