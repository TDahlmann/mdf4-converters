# MDF4 Iterator - Load Raw CAN Bus Data
This package lets you extract raw CAN bus data from the [CANedge](https://www.csselectronics.com/) MDF4 log files. The package can e.g. be used together with the `canedge_browser` and `can_decoder` modules.

---
### Key features
```
1. Easily extract raw CAN bus data from MDF4 files
2. Data can be loaded from a local path or e.g. from an S3 server path
3. The output can be an iterable or a pandas dataframe
4. Files can e.g. be loaded from lists generated by the canedge_browser package
5. Loaded CAN data can be decoded using the can_decoder package
6. No external dependencies
7. Faster bulk extraction using optional dependencies
```

---
### Installation
Use pip to install the `mdf_iter` module:
```
pip install mdf_iter
```
Optionally install `pandas` for fast bulk loading:
```
pip install pandas
```
---
### Dependencies
* `pandas` (optional)

---
### Platforms
Pre-built wheels are available for the following platforms:
* Linux: x86-64 (Python 3.5, 3.6, 3.7, 3.8)
* Windows: x86-64 (Python 3.7, 3.8)

Other platforms require manual compilation from source.

---
### Module usage examples
Below we open and iterate over the CAN records in a log file from local disk:
```
import mdf_iter

mdf_path = "00000001.MF4"

with open(mdf_path, "rb") as handle:
    mdf_file = mdf_iter.MdfFile(handle)
    record_iterator = mdf_file.get_can_iterator()
    
    for record in record_iterator:
        print(record)
```
Below we open a log file from an S3 server into a pandas dataframe:
```
import mdf_iter
import s3fs

fs = s3fs.S3FileSystem(
    key="<key>", secret="<secret>", client_kwargs={"endpoint_url": "<url>"}
)
mdf_path = "bucket_name/12345678/00000001/00000001-12345678.MF4"

with fs.open(mdf_path, "rb") as handle:
    mdf_file = mdf_iter.MdfFile(handle)
    df_raw = mdf_file.get_data_frame()

print(df_raw)
```
---
### Regarding data sources (local, S3 servers, ...)
The package can by default handle inputs in the form of:
* Python strings
* Python `Path` objects from `pathlib`
* File-like objects, obtained using `open(file_name, "rb")`

Any other data source can be adapted to work with the library via a wrapper. Simply create a custom wrapper which implements the `IFileInterface` class and pass it as the input to the `MdfFile` constructor.

For any custom objects which supports `read` and `seek`, a wrapper is supplied in the form of `FileInterface`. This can be used in conjunction with `fsspec` and similar projects. `MdfFile` handles this wrapper transparently.

```
fs = some_fsspec_filesystem_implementation
file_path = a_valid_fsspec_file_path

with fs.open(file_path, "rb") as handle:
    with mdf_iter.MdfFile(handle) as mdf_file:
        ...
```
In case it is necessary to create the wrapper manually, the usage is as below:
```
fs = some_fsspec_filesystem_implementation
file_path = a_valid_fsspec_file_path

with fs.open(file_path, "rb") as handle:
    wrapper = mdf_iter.FileInterface(handle)
    
    mdf_file = mdf_iter.MdfFile(wrapper)
    ...
```
For examples using fsspec to e.g. load data from an S3 server (for use with the [CANedge2](https://www.csselectronics.com/screen/product/can-lin-logger-wifi-canedge2/language/en), see the examples folder.

---
### Extraction methods

Data can be extracted either through an iterator, or in bulk using `pandas` if it is installed.
```
with mdf_iter.MdfFile("path") as mdf_file:
    # Using iterator.
    record_iterator = mdf_file.get_can_iterator()
    
    for record in record_iterator:
        ...
    
    # Using pandas.
    df_raw = mdf_file.get_data_frame()

print(df_raw)
```

---
### Extracting log file metadata
Metadata for a log file is accessible through `get_metadata()`. This returns a dictionary with string keys and dictionary values in the form of `MdfMetadataEntry`. These are also dictionaries, which expose the possible metadata from the blocks in the MDF file. The possible fields are:
* `description` - description of the field
* `name` - name of the field, also part of the key
* `read_only` - whether the field is marked as read only (has no effect)
* `unit` - associated unit for the field
* `value_raw` - the value as a string
* `value_type` - the value type

For further usage examples, see the `examples` folders.