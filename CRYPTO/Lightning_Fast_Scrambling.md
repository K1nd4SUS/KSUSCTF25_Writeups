The name reads LFS? and this is just a LFSR where you have to guess the last part of the key.
The LFSR works 8 bits per step and the key is the first 8 bytes of the produced keystream.
The plaintext is the flag and begins with "KSUS{" this gives 5 bytes of the key.
The remaining 3 bytes can be bruteforced.
The easy way is to exploit the low entropy of the rest of the flag (hex digits: 4 bits per byte), at most 4096 attempts are required.
It is indeed not possible to recover the passphrase but it doesn't matter

```Python
from lfs import *
from itertools import product

challenge = input("challenge > ")
e, hash = map(base64_decode, challenge.split('#'))
prefix = "KSUS{".encode()

for t in product("0123456789abcdef", repeat=3):
	missing_fragment = "".join(t)
	base_message = prefix + missing_fragment.encode()
	key_stream = [x ^ y for x, y in zip(e, base_message)]
	key = 0
	for x in reversed(key_stream):
		key <<= 8
		key |= x
	message = scramble(e, key)
	if hash == digest(message):
		flag = message.decode()
		break
else:
	assert False

print("key :", key)
print("flag :", flag)
```
