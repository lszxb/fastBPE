
# fastBPE

C++ implementation of [Neural Machine Translation of Rare Words with Subword Units](https://arxiv.org/abs/1508.07909), with Python API.

This is a fork of the original fastBPE, trying to build on Windows.

## Installation (Windows)

Environment:

Windows 10

gcc or clang from mingw-w64 or msys2

[mman-win32](https://github.com/alitrack/mman-win32)

cython

Compile with:
```
g++ -std=c++11 -pthread -O3 fastBPE/main.cc -IfastBPE -o fast -lmman -L<path to mman lib> -I<path to mman include>
```

## Usage:

### List commands
```
./fast
usage: fastbpe <command> <args>

The commands supported by fastBPE are:

getvocab input1 [input2]             extract the vocabulary from one or two text files
learnbpe nCodes input1 [input2]      learn BPE codes from one or two text files
applybpe output input codes [vocab]  apply BPE codes to a text file
applybpe_stream codes [vocab]        apply BPE codes to stdin and outputs to stdout
```

fastBPE also supports stdin inputs. For instance, these two commands are equivalent:
```
./fast getvocab text > vocab
cat text | ./fast getvocab - > vocab
```
But the first one will memory map the input file to read it efficiently, which can be more than twice faster than stdin on very large files. Similarly, these two commands are equivalent:
```
./fast applybpe output input codes vocab
cat input | ./fast applybpe_stream codes vocab > output
```
Although the first one will be significantly faster on large datasets, as it uses multi-threading to pre-compute the BPE splits of all words in the input file.

### Learn codes
```
./fast learnbpe 40000 train.de train.en > codes
```

### Apply codes to train
```
./fast applybpe train.de.40000 train.de codes
./fast applybpe train.en.40000 train.en codes
```

### Get train vocabulary
```
./fast getvocab train.de.40000 > vocab.de.40000
./fast getvocab train.en.40000 > vocab.en.40000
```

### Apply codes to valid and test
```
./fast applybpe valid.de.40000 valid.de codes vocab.de.40000
./fast applybpe valid.en.40000 valid.en codes vocab.en.40000
./fast applybpe test.de.40000  test.de  codes vocab.de.40000
./fast applybpe test.en.40000  test.en  codes vocab.en.40000
```

## Python API (Windows)

To install the Python API, do:
1. replace the `include_dirs` and `library_dirs` in `setup.py` with your path to mman-win32,
or put them in your system include and library path.

1. replace the vc runtime string "msvcr140" to "ucrtbase" in `<python root>\Lib\distutils\cygwinccompiler.py`

2. ```bash
   python setup.py build_ext --compiler mingw32
   python setup.py install
   ```

**Note:** The fastBPE extension will depend on the `libstdc++.dll` of msys2/mingw-w64, which may have conflicts with 
the lib in the conda environment. Make sure the python can find the correct version of `libstdc++.dll`, or a "DLL Load Failed" `ImportError` will be raised while importing fastBPE.  

Call the API using:

```python
import fastBPE

# bpe = fastBPE.fastBPE(codes_path, vocab_path)  # it's unable to read the vocab now
bpe = fastBPE.fastBPE(codes_path, '')
bpe.apply(["Roasted barramundi fish", "Centrally managed over a client-server architecture"])

>> ['Ro@@ asted barr@@ am@@ un@@ di fish', 'Centr@@ ally managed over a cli@@ ent-@@ server architecture']
```
