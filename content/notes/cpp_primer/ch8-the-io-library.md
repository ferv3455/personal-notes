---
title: Chapter 8 The IO Library
weight: 8
type: docs
---

## The IO Classes

- IO library types and headers (`wchar_t` versions begin with a `w`):

<img src="../attachments/Pasted image 20240923234256.png">

- **IO type objects cannot be copied or assigned.**
- Reading or writing an IO object changes its state.
- IO library condition states and ways to interrogate the state of flags:
	- `iostate` values are sets of bit flags representing different errors.

<img src="../attachments/Pasted image 20240923234644.png">

- Conditions that cause the buffer to be flushed:
	- The program completes normally (returns from `main`). **Buffers are not flushed if the program crashes.**
	- The buffer becomes full.
	- The buffer is flushed explicitly (`endl`: adds a newline, `flush`: no new data, `ends`: adds a null, etc.).
	- `unitbuf` is used to set the internal state to empty the buffer after each output: `cout << unitbuf / nounitbuf`. **`unitbuf` is set for `cerr` by default.**
	- If it is tied to another stream (`cin`, `cerr` are tied to `cout`), reading or writing on other streams will flush the tied output stream. Streams has two overloaded `tie` member functions: `tie()` returns the stream it's tied to, and `tie(s)` ties itself to `s`.

## File Input and Output

- Operations specific to `fstream`:

<img src="../attachments/Pasted image 20240924162822.png">

- `ifstream` and `ofstream` inherit from `istream` and `ostream`.
- If a call to `open` fails, `failbit` is set, and we can use `if (out)` to check if it succeeded.
- `fstream` objects are automatically closed if out of scope.
- File Modes (preceded with a scope `ofstream::`/`ifstream::`):
    - Default mode for `ofstream` is `out` and `trunc`.

<img src="../attachments/Pasted image 20240924162844.png">

## `string` Streams

- Operations specific to `stringstream` for quick input and output:

<img src="../attachments/Pasted image 20240924162901.png">

```cpp
PersonInfo info;            // create an object to hold this record's data 
istringstream record(line); // bind record to the line we just read 
record >> info.name;        // read the name
while (record >> word)      // read the phone numbers and store them
    info.phones.push_back(word);
```
