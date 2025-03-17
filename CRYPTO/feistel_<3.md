# K!nd4SUS CTF 2025

## Feistel <3

### Solution
The challenge involves a Feistel network used in CBC mode to encrypt the flag. In detail, the encryption function uses a 128-bit key $k$ that's split into 8 round keys $k_i$, and employs the F function:

$$F(B, k_{i}, N) = B + (65537^{k_{i}} \mod N) \mod 65536$$

We can summarize the operations in each round with these equations:

$$
\begin{align*}
    L_{i} &= L_{i-1} \oplus M_{i-1} \\ 
    M_{i} &= F(R_{i-1}, k_{i}, N)\\
    R_{i} &= M_{i-1} \oplus k_{i} \\
\end{align*}
$$

The oracle allows us to encrypt single blocks of 2 bytes, with a shortcut occurring when the block $L_{i}$ of the current round equals `0xffff`. Given this structure, we can extract each round key $k_i$ by crafting a plaintext that will produce `0xffff` at round $i$. We can then retrieve the key with:

$$M_{i-1} \oplus R_{i} = k_{i}$$

Having all the round keys $k_{j}$ for $j<i$, we can forge the necessary plaintext to trigger a shortcut at round $i$ by selecting a hex value where $L \oplus M = $ `0xffff`, and then decrypt from round $i-1$ to $0$ using the known round keys.

### Exploit
```python
from Crypto.Util.number import bytes_to_long, long_to_bytes
from pwn import *

def xor_bytes(bytes_a, bytes_b):
    return bytes(a ^ b for a, b in zip(bytes_a, bytes_b)).ljust(2, b'\x00')

def f(sub_block, round_key, modulus):
    sub_block = bytes_to_long(sub_block)
    round_key = bytes_to_long(round_key)
    res = (sub_block + pow(65537, round_key, modulus)) % (1<<17-1)
    return long_to_bytes(res).ljust(2, b'\x00')

def inv_f(sub_block, round_key, modulus):
    sub_block = bytes_to_long(sub_block)
    round_key = bytes_to_long(round_key)
    res = (sub_block - pow(65537, round_key, modulus)) % (1<<17-1)
    return long_to_bytes(res).ljust(2, b'\x00')

def decrypt_block(block, key, modulus, rounds=8):
    sub_block_1 = block[:2].ljust(2, b'\x00')
    sub_block_2 = block[2:4].ljust(2, b'\x00')
    sub_block_3 = block[4:].ljust(2, b'\x00')
    for i in range(rounds-1, -1, -1):
        round_key = key[i*2:i*2+2]
        prev_1 = xor_bytes(sub_block_1, xor_bytes(sub_block_3, round_key)) 
        prev_2 = xor_bytes(sub_block_3, round_key)
        prev_3 = inv_f(sub_block_2, round_key, modulus)
        sub_block_1 = prev_1
        sub_block_2 = prev_2
        sub_block_3 = prev_3
    return sub_block_1 + sub_block_2 + sub_block_3

def decrypt(ciphertext, key, modulus):
    iv = bytes.fromhex(ciphertext[:12])
    ciphertext = bytes.fromhex(ciphertext[12:])
    blocks = [ciphertext[i:i+6] for i in range(0, len(ciphertext), 6)] 
    res = b""
    for i in range(len(blocks)):
        block = decrypt_block(blocks[i], key, modulus)
        if i == 0: block = xor_bytes(block, iv)
        else: block = xor_bytes(block, blocks[i-1])
        res += block
    return res

def get_key(key, modulus, round):
    if key == b"":
        target = "0ffff000aaaa"
    else:
        target = decrypt_block(bytes.fromhex("0ffff000aaaa"), key, modulus, round-1).hex()
    conn.sendlineafter(b"> ", b"1")
    conn.sendlineafter(b"Enter your fantastic plaintext (in hex): ", target.encode())
    conn.recvuntil(b"Here it is: ")
    new_ct = conn.recvline().decode().strip()
    l = bytes.fromhex(target[:4])
    m = bytes.fromhex(target[4:8])
    r = bytes.fromhex(target[8:])
    for i in range(round-1):
        l_n = xor_bytes(l, m)
        m_n = f(r, key[i*2:i*2+2], modulus)
        r_n = xor_bytes(m, key[i*2:i*2+2])
        l = l_n
        m = m_n
        r = r_n
    round_key = xor_bytes(m, bytes.fromhex(new_ct[8:]))
    return round_key

conn = remote(...)
conn.recvuntil(b"flag = ")
FLAG = conn.recvline().decode().strip()
conn.recvuntil(b"N = ")
modulus = int(conn.recvline().decode().strip())
key = b""
for i in range(8):
    key += get_key(key, modulus, i+1)
print(key.hex())
print(decrypt(FLAG, key, modulus))
```