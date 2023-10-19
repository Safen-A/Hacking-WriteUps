# The Last Dance

Die Challenge `The Last Dance` bot mir eine spannende Einführung in den Verschlüsselungsalgorithmus ChaCha20. 
Es war das erste Mal, dass ich von dieser Verschlüsselungstechnik hörte, was den Spaßfaktor der Challenge erhöhte.

<img src="../screenshots/TheLastDanceFlag.png" width=800>

## ChaCha20: Eine kurze Einführung

**Was ist ChaCha20?** ChaCha20 ist ein Stromverschlüsselungsalgorithmus, der als Ersatz für ältere Algorithmen wie RC4 entworfen wurde.

**Grundlegende Eigenschaften:**

1. **Stromverschlüsselung:** Im Gegensatz zur Blockverschlüsselung, die Daten in festen Größenblöcken verschlüsselt, erzeugt die Stromverschlüsselung einen "Strom" von Pseudozufallsbits, der dann mit den zu verschlüsselnden Daten verknüpft wird.
2. **Sicherheit:** ChaCha20 gilt als sehr sicher und hat Widerstand gegenüber bekannten Angriffstechniken gezeigt.
3. **Geschwindigkeit:** Eine der Hauptstärken von ChaCha20 ist seine Geschwindigkeit, insbesondere in Software-Implementierungen.

**Anwendungsbereiche:** ChaCha20 wurde in verschiedenen Sicherheitsprotokollen und Standards übernommen. Ein bemerkenswertes Beispiel ist das TLS-Protokoll (Transport Layer Security), in dem ChaCha20 oft als eine bevorzugte Option für die Verschlüsselung verwendet wird, insbesondere wenn Hardwarebeschleunigung für andere Algorithmen wie AES nicht verfügbar ist.

**Wie funktioniert ChaCha20?**  
ChaCha20 nutzt ein Passwort und eine einmalige Nonce, um einen Schlüsselstrom zu erzeugen. Dieser Schlüsselstrom wird dann mit dem Klartext kombiniert, um den verschlüsselten Text zu erzeugen. 

Es ist **wichtig**, `bei jeder neuen Verschlüsselung eine neue Nonce zu verwenden.`

## Die Challenge im Detail

Nach dem Entpacken der bereitgestellten zip-Datei erhalten wir zwei Dateien:

- out.txt
- source.py

```txt
c4a66edfe80227b4fa24d431
7aa34395a258f5893e3db1822139b8c1f04cfab9d757b9b9cca57e1df33d093f07c7f06e06bb6293676f9060a838ea138b6bc9f20b08afeb73120506e2ce7b9b9dcd9e4a421584cfaba2481132dfbdf4216e98e3facec9ba199ca3a97641e9ca9782868d0222a1d7c0d3119b867edaf2e72e2a6f7d344df39a14edc39cb6f960944ddac2aaef324827c36cba67dcb76b22119b43881a3f1262752990
7d8273ceb459e4d4386df4e32e1aecc1aa7aaafda50cb982f6c62623cf6b29693d86b15457aa76ac7e2eef6cf814ae3a8d39c7
```

```python
from Crypto.Cipher import ChaCha20
from secret import FLAG
import os


def encryptMessage(message, key, nonce):
    cipher = ChaCha20.new(key=key, nonce=iv)
    ciphertext = cipher.encrypt(message)
    return ciphertext


def writeData(data):
    with open("out.txt", "w") as f:
        f.write(data)


if __name__ == "__main__":
    message = b"Our counter agencies have intercepted your messages and a lot "
    message += b"of your agent's identities have been exposed. In a matter of "
    message += b"days all of them will be captured"

    key, iv = os.urandom(32), os.urandom(12)

    encrypted_message = encryptMessage(message, key, iv)
    encrypted_flag = encryptMessage(FLAG, key, iv)

    data = iv.hex() + "\n" + encrypted_message.hex() + "\n" + encrypted_flag.hex()
    writeData(data)
```


Im Quellcode erkennen wir einen entscheidenden Fehler: Sowohl `encrypted_message` als auch `encrypted_flag` verwenden dieselbe Nonce (`iv`). Dies ermöglicht es uns, die Formel

`encrypted_message⊕encrypted_flag=message⊕FLAG`

zu nutzen. Umgeformt erhalten wir:

`encrypted_message⊕encrypted_flag⊕message=FLAG`

Mit dieser Erkenntnis können wir ein Python-Programm erstellen, welches die FLAG entschlüsselt:



```python
def xor_bytes(a, b):
    """XOR two byte strings and return the result."""
    return bytes(x ^ y for x, y in zip(a, b))

def decrypt_flag(ciphertext_message, ciphertext_flag, known_plaintext):
    """Decrypt the flag using the known-plaintext attack."""
    xor_ciphertexts = xor_bytes(ciphertext_message, ciphertext_flag)
    return xor_bytes(xor_ciphertexts, known_plaintext)

nonce = bytes.fromhex("c4a66edfe80227b4fa24d431")
ciphertext_message = bytes.fromhex("7aa34395a258f5893e3db1822139b8c1f04cfab9d757b9b9cca57e1df33d093f07c7f06e06bb6293676f9060a838ea138b6bc9f20b08afeb73120506e2ce7b9b9dcd9e4a421584cfaba2481132dfbdf4216e98e3facec9ba199ca3a97641e9ca9782868d0222a1d7c0d3119b867edaf2e72e2a6f7d344df39a14edc39cb6f960944ddac2aaef324827c36cba67dcb76b22119b43881a3f1262752990")
ciphertext_flag = bytes.fromhex("7d8273ceb459e4d4386df4e32e1aecc1aa7aaafda50cb982f6c62623cf6b29693d86b15457aa76ac7e2eef6cf814ae3a8d39c7")
known_plaintext = b"Our counter agencies have intercepted your messages and a lot " \
                  b"of your agent's identities have been exposed. In a matter of " \
                  b"days all of them will be captured"

# Decrypt the flag
decrypted_flag = decrypt_flag(ciphertext_message, ciphertext_flag, known_plaintext)
print(decrypted_flag.decode())
#HTB{und3r57AnD1n9_??????????????????????????}
```
