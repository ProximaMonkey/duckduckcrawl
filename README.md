# DuckDuckGo distributed crawler (DDC) prototype 

The purpose of this project is to prototype a distributed crawler for the DuckDuckGo search engine.

## Protocol

### Basic workflow
 * A client requests a list of domains to check for spam, the server answers with a list of domains
 * The server might also add in the response additional data to ask the client to upgrade itself or the page analysis component
 * The client does the analysis on the domains, and then sends the results back to the server
 * The client request another bunch of domains to check and so on

### Implementation
 * It's a classic REST API
 * To get a domain list the client sends a GET request, and to post the results it sends a POST request

### URL parameters: 
 * version : the protocol version which defines the XML response structure, it must be incremented when a change breaks client compatibility. The server must always handle all old protocol versions, to at least to tell the clients they must upgrade
 * pc_version : the version of the page processing binary component 

### XML response format
It contains one of these nodes immediately above the root:

 * 'upgrades' : can contain nodes to tell the client to upgrade its components (with URL to download the new version) 
 * 'domainlist' : the list of domains to check ('domain' nodes)

## Files

 * ddc_client.py : Code for a crawling worker
 * ddc_process.py : This file contains the code that simulates the binary component, currently it returns dumb results just to simulate
 * ddc_server.py : Code for the server that distributes the crawling work to the clients and gets the result from them
 * tests/single_client.sh : Bash script to do a small simulation by launching the server and connecting a client to it
 * tests/client_upgrade.sh : Bash script to simulate a client upgrade initiated by the server

## Dependencies

 * [Python 3.2](http://www.python.org/download/)

 * [httplib2 0.7+](https://code.google.com/p/httplib2/downloads/list)

I use the following Bash script to compile and install both on Ubuntu 10.10:

```bash
THREAD_COUNT=$([ $(which getconf) ] && getconf _NPROCESSORS_ONLN || grep -cE '^processor' /proc/cpuinfo)
TMP_DIR=$(mktemp -d /tmp/$(basename -- $0).XXXXXXXXXX)

# checkinstall
sudo apt-get -y install checkinstall

# python 3.2
sudo apt-get -y install build-essential libncursesw5-dev libreadline5-dev libssl-dev libgdbm-dev libc6-dev libsqlite3-dev tk-dev libbz2-dev
wget -P $TMP_DIR http://www.python.org/ftp/python/3.2.1/Python-3.2.1.tar.xz
tar -xJf $TMP_DIR/Python-3.2.1.tar.xz -C $TMP_DIR
pushd $TMP_DIR/Python-3.2.1 > /dev/null
./configure
make -j $THREAD_COUNT
sudo checkinstall --pkgname=python3.2 --pkgversion=3.2.1 --backup=no --deldoc=yes --fstrans=no --default make altinstall
popd > /dev/null
ln -sv /usr/local/bin/python3.2 /usr/local/bin/python3
ln -sv /usr/local/bin/pydoc3.2 /usr/local/bin/pydoc3

# httplib2
wget -P $TMP_DIR https://httplib2.googlecode.com/files/httplib2-0.7.2.tar.gz
tar -xzf $TMP_DIR/httplib2-0.7.2.tar.gz -C $TMP_DIR
pushd $TMP_DIR/httplib2-0.7.2 > /dev/null
sudo python3 setup.py install
popd > /dev/null

# cleanup
sudo rm -Rf $TMP_DIR
```

The code has only been tested on Linux but is fully OS neutral.
