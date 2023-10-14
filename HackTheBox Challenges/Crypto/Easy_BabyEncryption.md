# Easy_BabyEncryption

Die BabyEnryption Challenge ist eine sehr einfache Crypto Challenge. Hier zeige ich zwei Möglichkeiten diese zu lösen.
<img src="../screenshots/BabyEncryptionFlag.png" width=800>




Die Challenge besteht aus der chall.py

```python
# The chall.py file

import string
from secret import MSG

def encryption(msg):
    ct = []
    for char in msg:
        ct.append((123 * char + 18) % 256)
    return bytes(ct)

ct = encryption(MSG)
f = open('./msg.enc','w')
f.write(ct.hex())
f.close()
```
und der msg.enc Datei
```txt
6e0a9372ec49a3f6930ed8723f9df6f6720ed8d89dc4937222ec7214d89d1e0e352ce0aa6ec82bf622227bb70e7fb7352249b7d893c493d8539dec8fb7935d490e7f9d22ec89b7a322ec8fd80e7f8921
```

Gehen wir auf die wichtigsten informationen in der chall.py ein.

1. `from secret import MSG` hierbei wird aus dem secret Modul der Wert von MSG importiert. 
2. `def encryption(msg)`Hier wird msg übergeben und die eigentliche Verschlüsselungsmethode.
3. `ct = []` ersellt eine leere Liste um die Verschlüsselte Zeichen zu speichern
4. `for char in msg:` durchläuft jedes Zeichen in msg. 
5. `ct.append(123 * char +18) % 256)`. Dabei wird jedes Zeichen durchlaufen und mit 123 multipliziert, 18 addiert und anschließend Modulo 256 genommen. So hier sehen wir also, dass char ein int wert sein müsste. Darauf können wir schließen, das das Paket secret, in der Form vorgeht: Es wird ein String eingelesen, jeder char in dem String wird zu seinem dezimalen Unicode umgewandelt. Beispiel: `print(ord('A') # gibt 65 aus`.
6. Anschließend wird die liste in einen hexadezimal umgewandelt und in eine Datei gespeichert.


## Methode 1: Die Brute Force Methode

1. Wir konvertieren den msg.enc in eine Liste die jeweils aus zwei Zeichen besteht, und dannach in eine Dezimal Liste.
```python
hex_string = "6e0a9372ec49a3f6930ed8723f9df6f6720ed8d89dc4937222ec7214d89d1e0e352ce0aa6ec82bf622227bb70e7fb7352249b7d893c493d8539dec8fb7935d490e7f9d22ec89b7a322ec8fd80e7f8921"

# Wandele den Hex-String in eine Liste von Hex-Werten um
hex_list = [hex_string[i:i+2] for i in range(0, len(hex_string), 2)]

# Wandele die Hex-Liste in eine Dezimal-Liste um
decimal_list = [int(value, 16) for value in hex_list]

print("hex_list: ", hex_list)
print("decimal_list: ", decimal_list)

''' hex_list: ['6e', '0a', '93', '72', 'ec', '49', 'a3', 'f6', '93', '0e', 'd8', '72', '3f', '9d', 'f6', 'f6', '72', '0e', 'd8', 'd8', '9d', 'c4', '93', '72', '22', 'ec', '72', '14', 'd8', '9d', '1e', '0e', '35', '2c', 'e0', 'aa', '6e', 'c8', '2b', 'f6', '22', '22', '7b', 'b7', '0e', '7f', 'b7', '35', '22', '49', 'b7', 'd8', '93', 'c4', '93', 'd8', '53', '9d', 'ec', '8f', 'b7', '93', '5d', '49', '0e', '7f', '9d', '22', 'ec', '89', 'b7', 'a3', '22', 'ec', '8f', 'd8', '0e', '7f', '89', '21']
decimal_list: [110, 10, 147, 114, 236, 73, 163, 246, 147, 14, 216, 114, 63, 157, 246, 246, 114, 14, 216, 216, 157, 196, 147, 114, 34, 236, 114, 20, 216, 157, 30, 14, 53, 44, 224, 170, 110, 200, 43, 246, 34, 34, 123, 183, 14, 127, 183, 53, 34, 73, 183, 216, 147, 196, 147, 216, 83, 157, 236, 143, 183, 147, 93, 73, 14, 127, 157, 34, 236, 137, 183, 163, 34, 236, 143, 216, 14, 127, 137, 33]
'''
```

2. Wir suchen also zu jedem Wert in der decimal_list ein x sodass `123 * x +18) %256 == ct_value` ist. Dabei ist ct_value ein Wert in der decimal_list - Liste.

```python
def decrypt(ct_value):
    for x in range(256):
        if (123 * x + 18) % 256 == ct_value:
            return x
    return None  # Kein passendes x gefunden
```

3. Wir schreiben also eine for-schleife und durchlaufen die ganze decimal_list und entschlüsseln einzeln jedes Zeichen. Anschließend müssen wir nur noch die Werte in die Ascii darstellung umwandeln und als string speichern.
```python
decrypt_msg = []

# durchlaufe jeden Wert in decimal_list und entschlüssel diesen
for i in decimal_list:
    decrypt_msg.append(decrypt(i))


result = ''.join([chr(num) for num in decrypt_msg])

print("decrypt_msg :", decrypt_msg)
print("result :", result)

'''decrypt_msg : [84, 104, 51, 32, 110, 117, 99, 108, 51, 52, 114, 32, 119, 49, 108, 108, 32, 52, 114, 114, 49, 118, 51, 32, 48, 110, 32, 102, 114, 49, 100, 52, 121, 46, 10, 72, 84, 66, 123, 108, 48, 48, 107, 95, 52, 55, 95, 121, 48, 117, 95, 114, 51, 118, 51, 114, 115, 49, 110, 103, 95, 51, 113, 117, 52, 55, 49, 48, 110, 53, 95, 99, 48, 110, 103, 114, 52, 55, 53, 125]
result : Th3 nucl34r w1ll 4rr1v3 0n fr1d4y.
HTB{l00k_47?????????????????????????????????_c0ngr475}
'''
```



## Methode 2: Die Mathematische Methode


Diese Verschlüsselung nennt man in der Kryptografie `Das affine Kryptosystem` 


**Proposition**: Seien ![abinZmZ](https://latex.codecogs.com/svg.image?\dpi{200}a,%20b%20\in%20\mathbb{Z}/m\mathbb{Z) und sein ![fZmZZmZ](https://latex.codecogs.com/svg.image?\dpi{200}f:%20\mathbb{Z}/m\mathbb{Z}%20\to%20\mathbb{Z}/m\mathbb{Z})definiert durch ![eKaxb](https://latex.codecogs.com/svg.image.latex?e_K%28x%29%20%3D%20%28ax%2Bb%29%20%5Cmod%20m) Die Abbildung f ist genau dann injektiv, wenn ![ggtam](https://latex.codecogs.com/svg.image.latex?%5Ctext%7BggT%7D%28a%2Cm%29%20%3D%201) ist.




Es gibt also eine Abbildung die inverse zu unserer ist. Diese ist ![dKayb](https://latex.codecogs.com/svg.image.latex?d_K%28y%29%20%3D%20a%5E%7B-1%7D%28y-b%29%20%5Cmod%20m).

Wir setzen also unsere Zahlen ein:

e_K(x) = (123 *x +18) mod 256 

ggt(123,256) = 1

Wir müssen also das multiplikative inverse von 123 in Z/256Z finden. Dies können wir mit dem erweiterten Euklidischen Algorithmus finden. Wir suchen nach einem Online Tool, das Tool ist https://de.planetcalc.com/3311/ 

Also ist a^-1 = 179

Wir setzen ein.

d_K(y) = 179*(x-18) mod 256 = 179 *(x+238) mod 256

Wir schreiben das Programm dazu

```python
def affine_decrypt(ct_value):
    return (179*(ct_value+238) % 256)


decrypt_msg = []

# durchlaufe jeden Wert in decimal_list und entschlüssel diesen
for i in decimal_list:
    decrypt_msg.append(affine_decrypt(i))


result = ''.join([chr(num) for num in decrypt_msg])

print("decrypt_msg :", decrypt_msg)
print("result :", result)

'''decrypt_msg : [84, 104, 51, 32, 110, 117, 99, 108, 51, 52, 114, 32, 119, 49, 108, 108, 32, 52, 114, 114, 49, 118, 51, 32, 48, 110, 32, 102, 114, 49, 100, 52, 121, 46, 10, 72, 84, 66, 123, 108, 48, 48, 107, 95, 52, 55, 95, 121, 48, 117, 95, 114, 51, 118, 51, 114, 115, 49, 110, 103, 95, 51, 113, 117, 52, 55, 49, 48, 110, 53, 95, 99, 48, 110, 103, 114, 52, 55, 53, 125]
result : Th3 nucl34r w1ll 4rr1v3 0n fr1d4y.
HTB{l00k_47?????????????????????????????????_c0ngr475}
'''

```
