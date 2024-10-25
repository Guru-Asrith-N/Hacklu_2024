```
import secrets
from Crypto.Cipher import AES # pip install pycryptodome
import os
import string
import audio_engine

# flag = flag{y0u_
with open("../SECRET/flag.txt", "r") as fp:
    flag = fp.read().strip()
flag = flag.encode().hex()

if not os.path.isdir("snippets"):
    os.mkdir("snippets")

    for cnt,s in enumerate(string.digits + "abcdef"):
        os.system(f"ffmpeg -ss {cnt} -t 1 -i pump.opus snippets/{ord(s)}.voc")
sound_bites = []
for s in flag:
    out = audio_engine.extract_sound_data(f"snippets/{ord(s)}.voc")
    sound_bites.append(out)
audio_engine.create_voc_file("flag.voc", b"".join(sound_bites)) 


def encrypt_it():
    key = secrets.token_bytes(16)
    cipher = AES.new(key, AES.MODE_ECB)
    with open("flag.voc", "rb") as f:
        val = f.read()
    val += b"\x00" * (16 - len(val) % 16)
    with open("flag.enc", "wb") as f:
        f.write(cipher.encrypt(val))
encrypt_it()
```
The flag is read from the `flag.txt` file and stored in string format and then encoded in hexadecimal format   
The code creates a file named `snippets` if it doesn't exist and iterates over hexadecimal characters to extract 1 second of the audio file `pump.opus`   
The code iterates through each hexadecimal character in flag   
using `audio_engine.extract_sound_data` function it extracts audio data and appends to `sound_bites` list  
all audio snippets are gathered, joined and saved in a single file named `flag.voc`   
The flag is then encoded using the `encrypt_it` function which pads the file to a multiple of 16 and encrypts using AES_ECB    
the ciphertext is then saved in `flag.enc`    



#### the below solution was given in discord

```
import string
from collections import OrderedDict

OFFSET_ONE = 0x2eec0
OFFSET_TWO = 0x2eed0
datasets = []
with open("../PUBLIC/flag.enc", "rb") as fp0:
    all_data = fp0.read()

with open("../PUBLIC/flag.enc", "rb") as fp:
    fp.read(16*2)
    last_one = False
    while True:
        data = fp.read(16)
        if not data:
            break
        datasets.append(data)
        print(data.hex())
        fp.read(OFFSET_ONE if last_one else OFFSET_TWO)
        last_one = not last_one

out = []
uniq = list(OrderedDict.fromkeys(datasets))
placeholders = string.ascii_uppercase[6:]
for cnt, itm in enumerate(datasets):
    out.append(placeholders[uniq.index(itm)])
print("".join(out))

known_start = b"flag{y0u_".hex()
known_end = b"}".hex()

mappings = {k: b"" for k in list(set(out))}
for ind, s in enumerate(known_start):
    mappings[out[ind]] = s

# end flag
if mappings[out[-1]] == b"":
    mappings[out[-1]] = known_end[1]
if mappings[out[-2]] == b"":    
    mappings[out[-2]] = known_end[0]

###

new_attmpt = str("".join(out))
for k,v in mappings.items():
    if v != b"":
        new_attmpt = new_attmpt.replace(k, v)
print("replaced:")
print(new_attmpt)
print("guess:")

debugge = str(new_attmpt)
printable = ""
for c in range(0, len(debugge), 2):
    if all([x in string.hexdigits for x in debugge[c:c+2]]):
        printable += chr(int(debugge[c:c+2], 16))
    else:
        printable += "?"

print(printable)
```
The above code reads the `flag.enc` and prints it in `all_data` file    
Then the code is read for every 32(16*2) bytes by alternating between the offsets   
the unique blocks are stored in the order in which they first appear and are assigned a placeholder character(G to Z) and stored in `out`     
The beginning `flag{y0u_` and the ending `}` of the flag are known    
the code then replaces placeholders with known hexadecimal values    
It then converts the hexadecimal to ASCII creating a readable string that represents a guessed plaintext version of flag    
