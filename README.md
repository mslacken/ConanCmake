# ConanCmake

Simple `CMakeLists.txt` which creates conan recipes for the existing system libraries

# Usage

Clone the project and then run following command 
```
cmake -DLIBRARIES=re2 .
```
which will try to detect the requested library.
Now this library has to be imported into conan with following command
```
conan create ~/ConanExternal/re2/ --channel systemlib --user admin
```
and after the file `default` has been appended to the conan profile
the library should be ready to be consumed.
