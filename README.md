# C base KV render function

## Design considerations

The functionality creates embedded structures (structures within structures) in order to store temporary values, then renders the string at the end. 

Another equally valid approach would be to not store the temporary values, and simply render the string as each value is received. This would require you to calculate the length of the string and keep track of the position you are at. At some point, the functionality would need to be able to signal that no more values could be rendered. This would be more memory efficient as you would have no temp values to store and all conversion could happen on the stack.

You should be able to adapt the structures and specifically the render function to do exactly that. 
 
## Run from the command line

You can test the library by running the binary created with the library. 

Running `./build/data -h` or `./build/data --help` will give you the help screan

```shell
Construct a key-value string from individual arguments passed on the command line
Usage:
        ./build/data [-h | --help]: display this message
        ./build/data <key value list>
        Example: ./build/data name "Barry Robinson" company "Northrup Grumman" occupation "Lead cyber engineer"
        This wil produce the string : name=Barry Robinson,company=Northrup Grumman,ocupation=Lead cyber engineer
```

Supply a non deliniated list or argument in pairs and the program will construct a KV string from the arguments.


```shell
$ ./build/data name "Barry Robinson" company "Northrup Grumman" occupation "Lead cyber engineer"
String: "name=Barry Robinson,company=Northrup Grumman,ocupation=Lead cyber engineer"
$
```
## Functionality 

This is a simple library that tokens as `key` `value` pairs to be added into a structure, then rendered into a `KV` string. 

Currently, the library supports `int` and `char *` values, and has the following functions.

```c
data_t *create_data(size_t items);
data_err_t add_int_kv_value(data_t *data, const char *key, const int value);
data_err_t add_str_kv_value(data_t *data, const char *key, const char *value);
data_err_t render(data_t *data);
void teardown(data_t *data);
const char * get_string_value_for_err(data_err_t err);
```

Extending supported types is a matter of implementing a function to convert the type to a string, and then calling `add_str_kv_value` on the `data` object.

## Using the library

```c
data_t * data = create_data(3);
data_err_t err = add_int_kv_value(data,"key",123);
assert(err == OK);
err = add_str_kv_value(data,"other_key","some value");
assert(err == OK);

err = add_int_kv_value(data,"key_1234",564092345);
assert(err == OK);

err = render(data);
assert(err == OK);

teardown(data);
```

In this instance, `render` will fill out the `data->string` with the value `"key=123,other_key=some value,key_1234=564092345"`. 


At the end of the life of the object `teardown` must be called to free memory allocated by the library.

## Runing benchmarks

To run `benchmarks` you will need to build and install the `Google Benchmark` framework. 

```shell
# Check out the library.
$ git clone https://github.com/google/benchmark.git
# Go to the library root directory
$ cd benchmark
# Make a build directory to place the build output.
$ cmake -E make_directory "build"
# Generate build system files with cmake, and download any dependencies.
$ cmake -E chdir "build" cmake -DBENCHMARK_DOWNLOAD_DEPENDENCIES=on -DCMAKE_BUILD_TYPE=Release ../
# or, starting with CMake 3.13, use a simpler form:
# cmake -DCMAKE_BUILD_TYPE=Release -S . -B "build"
# Build the library.
$ cmake --build "build" --config Release 
sudo cmake --build "build" --config Release --target install
```

## Runing Tests

To run `benchmarks` you will need to build and install the `Google Benchmark` framework.

```shell
git clone https://github.com/google/googletest.git -b v1.13.0
cd googletest        # Main directory of the cloned repository.
mkdir build          # Create a directory to hold the build output.
cd build
cmake ..             # Generate native build scripts for GoogleTest.

make
sudo make install
```
## TODO

1. Rename functions and structures to avoid namespace pollution.
2. Extend the functionality so that the structure can be reset.
   1. This should allow the reuse of already allocated memory, so 
   must track what has been allocated as well as what has been used
   and reset other variables to point to the start of usable space.
   2. It will also need to clear existing entries so that their memory 
   footprint (their content) is erased.
3. Extend the library so that the structure can be enlarged.
4. Document structures and functions, this should include (see example above `create_data`)
   1. required arguments
   2. return values
