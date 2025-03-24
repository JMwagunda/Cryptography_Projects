# Breaking Many-Time Pad Encryption
## Project Overview
This project demonstrates how to break ciphertexts encrypted using the same stream cipher key (a many-time pad vulnerability). The attack leverages the properties of XOR operations and statistical analysis to recover the plaintext without knowing the key.

## Key Features
* Hex Decoding & Byte-Level Analysis: Processes ciphertexts as raw bytes for precise XOR operations.

* Statistical Key Recovery: Uses frequency analysis to guess the most probable key bytes.

* Handling Variable-Length Ciphertexts: Works even if ciphertexts have different lengths.

* Target Decryption: Recovers the secret message from the target ciphertext.

## Thought Process & Methodology

### 1. Understanding the Many-Time Pad Vulnerability
* Problem: Reusing the same key (K) for multiple plaintexts (P₁, P₂, ..., Pₙ) means:
  
    C₁ = P₁ ⊕ K
    
    C₂ = P₂ ⊕ K
    
    ...
    
    Cₙ = Pₙ ⊕ K

 * Exploit: XORing two ciphertexts cancels out the key:
   
    C₁ ⊕ C₂ = P₁ ⊕ P₂
   
 * Insight: If we know (or guess) part of one plaintext (P₁), we can recover parts of others.

### 2. Statistical Attack on XOR-Based Encryption
* Key Observation:
  - Space (0x20) XOR [a-zA-Z] flips the case (e.g., 'A' ^ ' ' = 'a').
  - If a plaintext byte is a space, XORing with another plaintext byte reveals information about the other plaintext.
* Approach:
  - For each ciphertext position, guess possible key bytes.
  - Count how often a key byte produces valid plaintext (spaces or letters).
  - The most probable key byte is selected.

### 3. Handling Odd-Length & Variable-Length Ciphertexts
* Hex Decoding:
  - All ciphertexts must be valid hex (even-length), or binascii.unhexlify() fails.
  - Ensures proper byte alignment.

* Dynamic Key Recovery:
  - The key length is set to the longest ciphertext (max_len).
  - For shorter ciphertexts, only available bytes contribute to key recovery.
  - Missing positions are filled with placeholders (?).
* Robust Decryption:
  - If the target ciphertext is longer than the recovered key, extra bytes are marked as unknown (?).
  - If shorter, only the required key bytes are used.

# Unique Solutions & Edge Cases
### 1. Dealing with Different-Length Ciphertexts
  * Problem: Ciphertexts may have varying lengths, making key recovery tricky.
    ### Handling Variable-Length Ciphertexts
    ```python
    max_len = max(len(ct) for ct in cts)  # Longest ciphertext determines key length
    key = bytearray(max_len)
    
    for i in range(max_len):
        for ct in cts:
            if i < len(ct):  # Only process available bytes
                ct_byte = ct[i]
                # Key recovery logic...
    ```
  * Solution:
    - Only consider positions where a ciphertext has a byte (if i < len(ct)).
    - The key is built incrementally, using available data.

### 2. Statistical Weighting
  * Problem: Some key guesses are more likely than others.

  * Solution:
    - Spaces (0x20) are given full weight (counts[possible_key] += 1).
    - Letters (a-z, A-Z) are given partial weight (counts[possible_key] += 0.5).
    - The most probable key byte is selected (max(counts.items(), key=lambda x: x[1])).

### 3. Handling Incomplete Key Recovery
  * Problem: Some key bytes may remain unknown if ciphertexts are too short.
    ### Robust Decryption
    ```python
    for i in range(len(target_ct)):
        if i < len(key):
            plaintext.append(target_ct[i] ^ key[i])
        else:
            plaintext.append(ord('?'))  # Placeholder for missing key bytes
    ```
  * Solution:
    - Missing key bytes default to 0 (due to bytearray(max_len)).
    - During decryption, undecryptable bytes are replaced with ?.

## How to Use the Code
  1. Input: Provide ciphertexts as hex strings (even-length).
  2. Execution:
     ```python
     python3 many_time_pad_attack.py
     ```
  3. Output:
     - The most probable plaintext for the target ciphertext.
     - Non-printable bytes are replaced with ..
       
## Example Output
    Decrypted message: Whxn using a ~tream cipher, never use the key more than once
    Secret message: The secuettmessage&is: Whxn using a ~tream cipher, never use the key more than once  
## Conclusion
This project successfully breaks a many-time pad encryption by exploiting XOR properties and statistical analysis. It handles variable-length ciphertexts and efficiently recovers plaintext without knowing the key.
