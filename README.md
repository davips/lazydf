![test](https://github.com/safeserializer/safeserializer/workflows/test/badge.svg)
[![codecov](https://codecov.io/gh/safeserializer/safeserializer/branch/main/graph/badge.svg)](https://codecov.io/gh/safeserializer/safeserializer)
<a href="https://pypi.org/project/safeserializer">
<img src="https://img.shields.io/github/v/release/safeserializer/safeserializer?display_name=tag&sort=semver&color=blue" alt="github">
</a>
![Python version](https://img.shields.io/badge/python-3.10-blue.svg)
[![license: GPL v3](https://img.shields.io/badge/License-GPLv3_%28ask_for_options%29-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![API documentation](https://img.shields.io/badge/doc-API%20%28auto%29-a0a0a0.svg)](https://safeserializer.github.io/safeserializer)


# safeserializer - Serialization of nested objects to binary format 
An alternative to pickle, but may use pickle if safety is not needed.

Principle: Start from the simplest and safest possible and try to be fast.
* try orjson
  * `dict`, `str`, `int`, etc
* try bson
  * standard types accepted by mongodb
* convert bigints to str
* try to serialize as raw numpy bytes
  * ndarray, pandas homogeneous Series/DataFrame
* try parquet
  * pandas ill-typed Series/DataFrame
* resort to pickle if allowed (`unsafe_fallback=True`)
* resort to dill if allowed (`ensure_determinism=False`).



## Python installation
### from package
```bash
# Set up a virtualenv. 
python3 -m venv venv
source venv/bin/activate

# Install from PyPI
pip install safeserializer
```

### from source
```bash
git clone https://github.com/safeserializer/safeserializer
cd safeserializer
poetry install
```

### Examples
**Basic**
<details>
<p>

```python3
from safeserializer import unpack, pack
from pandas import Series as S, DataFrame as DF

df = DF({"a": ["5", "6", "7"], "b": [1, 2, 3]}, index=["x", "y", "z"])
complex_data = {"a": b"Some binary content", ("mixed-types tuple as a key", 4): 123, "df": df}
print(complex_data)

dump = pack(complex_data, ensure_determinism=False, unsafe_fallback=False)
print(dump)
"""
{'a': b'Some binary content', ('mixed-types tuple as a key', 4): 123, 'df':    a  b
x  5  1
y  6  2
z  7  3}
b'00lz4__\x04"M\x18h@h\x0b\x00\x00\x00\x00\x00\x00\x94\x95\x06\x00\x00\xf1*00dicB_a\x0b\x00\x00\x0530306a736f6e5f226122\x00\x13\x00\x00\x00\x00Some binary content.\x00\xd27475706c5f480\x01\x00b45f004\x0c\x00\x8000530002\x05\x00\x01\x02\x00\rb\x00\xf5\x07d697865642d74797065732X\x00`652061\x12\x00\xf4\x0461206b6579220531000r\x00\x0bV\x00\x113}\x00 \x00\n\xb8\x00\xa100json_123\xaf\x00\t\xdd\x00p46622\x00b(\x00\xf0\x1100prqd_PAR1\x15\x04\x15\x1e\x15"L\x15\x06\x15\x00\x12\x00\x00\x0f8\x01\x00\x00\x005\x05\x00\x106\x05\x00\xf667\x15\x00\x15\x14\x15\x18,\x15\x06\x15\x10\x15\x06\x15\x06\x1c6\x00(\x017\x18\x015\x00\x00\x00\n$\x02\x00\x00\x00\x06\x01\x02\x03$\x00&\x94\x01\x1c\x15\x0c\x195\x10\x00\x06\x19\x18\x01a\x15\x02\x16\x06\x16\x84\x01\x16\x8c\x01&F&\x085\x00\xe0\x19,\x15\x04\x15\x00\x15\x02\x00\x15\x00\x15\x10\x15?\x00d\x15\x04\x150\x15.\x7f\x00p\x18\x04\x01\x00\t\x01<\x19\x00\x00\x02\x00\x10\x03\x05\x00 \x00\x00.\x00\t\x85\x00$\x18\x08\x1a\x00 \x18\x08\xa6\x00\x00\x02\x00?\x16\x00(\x16\x00\x00\x0c\xa7\x00T\xe2\x03\x1c\x15\x04\xa7\x00\x11b\xa7\x00\xbf\xda\x01\x16\xdc\x01&\xd0\x02&\x86\x02Y\x00\x19\x0f\xcb\x00\x02\rJ\x01\x10x\x9f\x00\x10y\x05\x00\x1fzJ\x01\x01Lz\x18\x01x\xa3\x00&\xa8\x06J\x01\xf1\x03\x11__index_level_0__\xb3\x00\x02Z\x01Q\xda\x05&\x9c\x05\x91\x01\x01G\x00\x0f\x91\x00\x01\xf0\x0b\x19L5\x00\x18\x06schema\x15\x06\x00\x15\x0c%\x02\x18\x01a%\x00L\x1cV\x01\x10\x04\x0e\x00\x12b\x16\x00\x0ej\x00\x03&\x00o\x16\x06\x19\x1c\x19<\xe0\x01&\x0fr\x01J\x0f,\x018\xb0\x16\xe2\x03\x16\x06&\x08\x16\xf4\x03\x14\xdc\x01\xd2\x18\x06pandas\x18\xd5\x04{"\x83\x01\xcdcolumns": ["\x97\x01R"], ""\x00\x02\xb2\x01\x11e)\x00\xfa\x07{"name": null, "field_\x14\x00\x02i\x00@_typ)\x00\xf5\x02"unicode", "numpy\x19\x00`object\x18\x00\xf6\x12metadata": {"encoding": "UTF-8"}}\x8d\x00\n\x86\x00 "a?\x00\t\x85\x00\x02\x13\x00\x0f\x84\x00*\x00\xdc\x005}, \xec\x00."bf\x00\x01\x13\x00\x0bf\x00Pint64+\x00\n\xe8\x00\x05\x17\x00\x07\xe7\x00\x0cc\x00\x00\x10\x00\x0cO\x01\x0f\x95\x01\x00\x0et\x00\x0f^\x01\x1b\x00g\x00\x02M\x01areatorq\x01plibraryp\x01ppyarrow\xc4\x00pversion\x16\x00\x8611.0.0"}~\x00\x08\x1d\x00\xf2\x00.5.3"}\x00\x18\x0cARROW:\x9c\x03@\x18\x98\t/\x01\x00\x822gDAAAQA\x01\x00\xf1\x00KAA4ABgAFAAgACg\x15\x00)BB \x00\x10w\x15\x00\x15E \x002IwC\x10\x00\x04F\x00\x01 \x00\x10I\x08\x00\x11B\x08\x00\x01E\x00\x10I \x00\x10E\x05\x00\xf4GAYAAABwYW5kYXMAAFUCAAB7ImluZGV4X2NvbHVtbnMiOiBbIl9faW5kZXhfbGV2ZWxfMF9fIl0sICJjb2x1bW5$\x00\xf0\x01lcyI6IFt7Im5hbWUD\x00\xf0\x11udWxsLCAiZmllbGRfbmFtZSI6IG51bGwH\x00\x03\x90\x00aNfdHlw\x1c\x00\xf0\x0eCJ1bmljb2RlIiwgIm51bXB5X3R5cGX\x00\xa2Aib2JqZWN0 \x00\xf0\x0c1ldGFkYXRhIjogeyJlbmNvZGluZ\x94\x00\xe0CJVVEYtOCJ9fV0t\x00\x04\xbc\x00\x10z0\x00EW3si\x98\x002CJhT\x00\x84ZpZWxkX2\xcc\x00PAiYSI<\x00\x0f\xb0\x00>\xa9bnVsbH0sIH\x88\x00\x1fi\x88\x00\x06\x1fi\x88\x00\x06qpbnQ2NC \x00\xd0udW1weV90eXBl\xe8\x00\x00\xe8\x01?dDY4\x01\x02\x0f\x84\x00\x02\x06\xa4\x01\xc2maWVsZF9uYW1L\x00\x0f\x1c\x02\x05\x00\xb0\x01\x97nBhbmRhc1|\x00PnVuaW\x94\x01 Ui\x10\x02xbnVtcHl\xf4\x01\x82vYmplY3Q \x00\xa5WV0YWRhdGEH\x02\x04\xbc\x01\x80cmVhdG9y\xd4\x00\xc0eyJsaWJyYXJ5\x10\x00\xb1InB5YXJyb3cH\x00\xf9\x0bdmVyc2lvbiI6ICIxMS4wLjAifS\xa8\x00\x912ZXJzaW9uD\x00\xa0jEuNS4zIn06\x03\x00\xa4\x03!Ah\n\x00\x01`\x03\x01K\x03PmP///\x0f\x00!QU\xc0\x03\x10J \x00\x05\xcb\x030AAE\x16\x00\x1fF$\x01\x03\x00 \x00\x01@\x00`8z///8\xc3\x03\x11CU\x00\x10BQ\x00\x11A\x0b\x00\x02\x02\x00\x00\x0b\x00 Bi\x0c\x00@CAAM\xd0\x03"Bw\xd0\x03\x01\x02\x00\x11U\x06\x00\xc2QABQACAAGAAc\xad\x00\x12B:\x00\x01\x02\x00\x03\xa0\x00\x12G\r\x00\x01\x06\x00\x01\x02\x00\x00\xa0\x00\x10G`\x00\x00p\x001QAB\x15\x00\xf1\x02==\x00\x18 parquet-cpp-\xf1\x04\x13 \xd1\x04\x12 \xeb\x04R\x19<\x1c\x00\x00\x03\x00\xa0\x00t\x08\x00\x00PAR1\x00\x00\x00\x00\x00'
"""
```

```python3

obj = unpack(dump)
print(obj)
"""
{'a': b'Some binary content', ('mixed-types tuple as a key', 4): 123, 'df':    a  b
x  5  1
y  6  2
z  7  3}
"""
```


</p>
</details>



## Grants
This work was partially supported by Fapesp under supervision of
Prof. André C. P. L. F. de Carvalho at CEPID-CeMEAI (Grants 2013/07375-0 – 2019/01735-0).
