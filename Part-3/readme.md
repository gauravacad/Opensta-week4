# Part 3 – Generate Timing Graphs with OpenSTA
## Objective
- The purpose of this README is to document the OpenSTA timing analysis workflow, including installation, running example analyses, and comparing timing slacks for different designs.
- This task focuses on understanding timing behavior, slack calculation, and critical path evaluation using OpenSTA.

## Installation OpenSTA with dependencies 

### CUDD Installation
``` bash
$ git clone https://github.com/ivmai/cudd.git
$ cd cudd
$ aclocal -I . ; autoheader; autoconf; automake --add-missing -c
$ ./configure --enable-obj
$ make
$ make check
...
============================================================================
Testsuite summary for cudd 3.0.0
============================================================================
# TOTAL: 30
# PASS:  30
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================

# lastly
sudo make install 
```
<img width="829" height="720" alt="image" src="https://github.com/user-attachments/assets/b9ac5aeb-f4c9-470b-923c-6a9575f68008" />

### Install Swig

``` bash
sudo apt install swig
```

<img width="814" height="494" alt="image" src="https://github.com/user-attachments/assets/4841f05c-a47d-47cf-ae16-c7b207393396" />


### Installation of OpenSTA
```
sudo apt-get update
sudo apt-get install build-essential tcl-dev tk-dev cmake git libeigen3-dev autoconf m4 perl automake 

git clone https://github.com/The-OpenROAD-Project/OpenSTA.git
cd OpenSTA
mkdir build
cd build
cmake .. -DUSE_CUDD=ON -DCUDD_DIR=$HOME/cudd
make
sudo make install
```

sta

```
