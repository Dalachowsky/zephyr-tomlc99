# zephyr-tomlc99

TOML in c99; v1.0 compliant.

Wrapper around tomlc99 library for Zephyr RTOS.

## Usage

Parsing simple config in Zephyr RTOS

```c
const char *config = "foo = \"bar\"";
uint8_t heap[1024];
toml_parser_get(&heap, sizeof(heap), K_NO_WAIT);
if (toml_parser_get(&heap, 1024, K_NO_WAIT) != 0) {
    LOG_ERR("Error");
    return -1;
}
toml_table_t *table = toml_parse_string(config);
toml_datum_t foo = toml_string_in(table, "foo");
LOG_INF("foo: %s", foo.u.s);

toml_parser_free(table);
```

#### Heap

Since toml is mostly used for config files I have decided implement a "heap buffer".
tomlc99 makes use of `malloc()` and `free()` functions. I did not want to reserve
heap space for parsing a config file that is done once or twice. Instead i added the `heap`
parameter to `toml_parser_get` which is used to initialize temporary heap space.

#### Limitations

Only one file can be parsed at a time. This is due to my override of malloc and free functions.
In some time I might try making my implementation nicer, this one was supposed to be quick and easy.


#### Accessing Table Content

TOML tables are dictionaries where lookups are done using string keys. In
general, all access functions on tables are named `toml_*_in(...)`.

In the normal case, you know the key and its content type, and retrievals can be done
using one of these functions:
```c
toml_string_in(tab, key);
toml_bool_in(tab, key);
toml_int_in(tab, key);
toml_double_in(tab, key);
toml_timestamp_in(tab, key);
toml_table_in(tab, key);
toml_array_in(tab, key);
```

You can also interrogate the keys in a table using an integer index:
```c
toml_table_t* tab = toml_parse_string(...);
for (int i = 0; ; i++) {
    const char* key = toml_key_in(tab, i);
    if (!key) break;
    printf("key %d: %s\n", i, key);
}
```

#### Accessing Array Content

TOML arrays can be deref-ed using integer indices. In general, all access methods on arrays are named `toml_*_at()`.

To obtain the size of an array:
```c
int size = toml_array_nelem(arr);
```

To obtain the content of an array, use a valid index and call one of these functions:
```c
toml_string_at(arr, idx);
toml_bool_at(arr, idx);
toml_int_at(arr, idx);
toml_double_at(arr, idx);
toml_timestamp_at(arr, idx);
toml_table_at(arr, idx);
toml_array_at(arr, idx);
```

#### toml_datum_t

Some `toml_*_at` and `toml_*_in` functions return a toml_datum_t
structure. The `ok` flag in the structure indicates if the function
call was successful. If so, you may proceed to read the value
corresponding to the type of the content.

For example:
```
toml_datum_t host = toml_string_in(tab, "host");
if (host.ok) {
	printf("host: %s\n", host.u.s);
	free(host.u.s);   /* FREE applies to string and timestamp types only */
}
```

** IMPORTANT: if the accessed value is a string or a timestamp, you must call `free(datum.u.s)` or `free(datum.u.ts)` respectively after usage. **

## Building and installing

A normal *make* suffices. You can also simply include the
`toml.c` and `toml.h` files in your project.

Invoking `make install` will install the header and library files into
/usr/local/{include,lib}.

Alternatively, specify `make install prefix=/a/file/path` to install into
/a/file/path/{include,lib}.

## Testing

To test against the standard test set provided by toml-lang/toml-test:

```sh
% make
% cd test1
% bash build.sh   # do this once
% bash run.sh     # this will run the test suite
```


To test against the standard test set provided by iarna/toml:

```sh
% make
% cd test2
% bash build.sh   # do this once
% bash run.sh     # this will run the test suite
```