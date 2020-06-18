
#   :pushpin:   Building ZMQ C++ for linux!

###   :books:  Used resources :
* this nice [Chinese website](https://www.youtube.com/watch?v=DRH-EaIhOlc&list=PLS1lqxOwNjOaEFHEhbU_5uUZwrUquKTwZ) *(translate it to english for better undrstanding of code snippets)*    :cn:
* Readme of **Zmqpp** repo on [Github](https://github.com/zeromq/zmqpp) 
* code samples from [Zeromq Documentation](http://zguide.zeromq.org/page:chapter1)

**1.** Follow the official instructions to download the three packages :
```bash
$ git clone git://github.com/jedisct1/libsodium.git
$ git clone git://github.com/zeromq/libzmq.git
$ git clone git://github.com/zeromq/zmqpp.git
```
--> *then you will need to go build them in order*

----
**2.**  Build **libsodium** :

:arrow_forward: official script but didn't work with me    :no_entry:
```bash
cd libsodium
./autogen.sh 
```
`**unsolved error here : a development environment was not created`    :lock:
```bash
./configure && make check 
sudo make install 
sudo ldconfig
cd ../
```
:arrow_forward:  what really worked :+1:
```bash
sudo apt-get update
sudo apt-get install libsodium-dev
```
---
**3.** Build **libzmq** :
```bash
cd libzmq
sudo apt-get install autoconf autogen 
```
:arrow_forward:  `the previous line solved some errors`   :heavy_check_mark:
```bash
./autogen.sh 
./configure --with-libsodium && make 
```
:arrow_forward:  read :one:, :two: and :three: for possible errors   
```bash
sudo make install
sudo ldconfig
cd ../
```
:one:  possible error after running autogen.sh: 
>autogen.sh: error: could not find libtool.  libtool is required to run autogen.sh

#### `Cause:`
autogen.h has a check for lib-tool binary.

#### `Solution:`
install lib tool binary.
```bash 
sudo apt-get install libtool-bin
```
:two:  an error here might arise after running configure.sh: 
>checking for working volatile... yes
checking for sodium... no
./configure: line 421: test: then: integer expression expected
configure: error: run
./configure: line 310: return: then: numeric argument required
./configure: line 320: exit: then: numeric argument required

#### `Cause:`
address of the libsodium dynamic library cannot be viewed during the call.

#### `Solution:`
change in configurations file.
```bash 
echo "/usr/local/lib">>/etc/ld.so.conf
```
check the **edit**
```bash 
cat /etc/ld.so.conf
```
output should be like : 
>**include ld.so.conf.d/*.conf**
**/usr/local/lib**

:three:  another possible error after running configure.sh : 
>configure.ac:28: error: possibly undefined macro: AC_SUBSTIf this token and others are legitimate, please use m4_pattern_allow.See the Autoconf documentation.configure.ac:72: error: missing some pkg-config macros (pkg-config package)

#### `Cause:`
pkg-config package is missing.

#### `Solution:`
installing the package using the following commands.
```bash 
sudo apt-get update -y
sudo apt-get install -y pkg-config
```
---
**4.** Build **zmqpp**
```bash
cd zmqpp
sudo apt-get install libzmq3-dev 
```
:arrow_forward: `the previous line solved some errors`   :heavy_check_mark:
```bash
make
sudo apt-get install libboost-all-dev 
```
:arrow_forward: `the previous line solved some errors`   :heavy_check_mark:
```bash
make client
make check
sudo make install
make installcheck
```
---
**5**. now try to compile a file using ZMQ with this line :
```bash
g++ server.cpp -o server /usr/local/lib/libzmqpp.a  -lzmq
```
`/usr/local/lib/libzmqpp.a` & `-lzmq` parts are very important for the code to see the library so don't remove it 

----
**7.** run server and client on different terminals :
```ruby
$ ./client 
Connecting to hello world server…
Sending Hello 0…
Received World 0
Sending Hello 1…
Received World 1
Sending Hello 2…
Received World 2
Sending Hello 3…
Received World 3
Sending Hello 4…
Received World 4
``` 
```ruby
$ ./server
Received Hello
Received Hello
Received Hello
Received Hello
Received Hello
```

