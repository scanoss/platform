# SCANOSS Platform Deployment Guide

## Requirements

The SCANOSS Platform runs on Linux, but it could certainly be ported to Windows or OSX as it is written entirely in C. This guide is written for Ubuntu Server 20.04 LTS. Package names may vary depending on the Linux distribution used.

Before installing, make sure all software requirements are in place:

```
sudo apt install build-essential unzip libssl-dev zlib1g-dev p7zip-full unrar-free
```

# Installation from sources

## Installing the LDB

LDB is the database engine used for the Knowledge Base. The LDB is required for the engine and minr. The source code can be downloaded and compiled as follows:

```
wget -O ldb.zip https://github.com/scanoss/ldb/archive/master.zip
unzip ldb.zip
cd ldb-master
make
sudo make install
sudo mkdir /var/lib/ldb
sudo chown user.user /var/lib/ldb   # Replace user.user with your user and group
cd ..
ldb
```

The ldb prompt shows the LDB has been properly installed. You can simply quit as follows: 

```
Welcome to LDB 2.02
Use help for a command list and quit for leaving this session

ldb> quit
```


## Installing minr

Minr is the command-line tool used for downloading and indexing source code. The source code can be downloaded and compiled as follows:

```
wget -O minr.zip https://github.com/scanoss/minr/archive/master.zip
unzip minr.zip
cd minr-master
make
sudo make install
cd ..
minr -v
```

The last command should show the installed version of minr.

### Maximum number of open files

Minr requires a high limit on amount of simultaneous open files. This is configured as follows:

```
echo "ulimit -n 70000" > ~/.bash_profile
```

Restart your bash session before you try minr

## Installing the Inventory Engine

The SCANOSS Inventory Engine a command-line tool used for comparing a file or directory against the Knowledgebase. The source code can be downloaded and compiled as follows:

```
wget -O engine.zip https://github.com/scanoss/engine/archive/master.zip
unzip engine.zip
cd engine-master
make
sudo make install
cd ..
scanoss -v
```

The last command should show the installed version of the SCANOSS Inventory Engine.

# Testing minr and the Inventory Engine

The following examples show the entire process for downloading an OSS component, importing it into the Knowledge Base and performing a scan against it using the SCANOSS Open Source Inventory Engine:


## URL mining

URL mining is the process of downloading a component, expanding the files and saving component, metadata and original sources for snippet mining.

```
$ minr -d scanoss,webhook,1.0 -u https://github.com/scanoss/webhook/archive/1.0.tar.gz
Downloading https://github.com/scanoss/webhook/archive/1.0.tar.gz
$
```

A mined/ directory is created with the downloaded metadata. This includes component and file metadata and source code archives which are kept using the .mz archives, specifically designed for this purpose.

## Snippet mining

Snippet mining is a second step, where snippet information is mined from the .mz archives. This is achieved with the -z parameter.

```
$ minr -z mined
mined/sources/146a.mz
...
$
```

A mined/snippets directory is created with the snippet wfp fingerprints extracted from the .mz files located in mined/sources

## Data importation into the LDB

```
$ minr -i mined/
$
```

The LDB is now loaded with the component information and a scan can be performed.

## Scanning against the LDB Knowledge Base

The following example shows an entire component match:

```
$ scanoss 1.0.tar.gz
{
  "1.0.tar.gz": [
    {
      "id": "component",
      "elapsed": "0.001139s",
      "lines": "all",
      "oss_lines": "all",
      "matched": "100%",
      "vendor": "scanoss",
      "component": "webhook",
      "version": "1.0",
      "latest": "1.0",
      "url": "https://github.com/scanoss/webhook/archive/1.0.tar.gz",
      "file": "all",
      "size": "N/A",
      "dependencies": [],
      "licenses": []
    }
  ]
}
$
```

The following example shows an entire file match:

```
$ scanoss webhook-1.0/scanoss/github.py
{
  "webhook-1.0/scanoss/github.py": [
    {
      "id": "file",
      "elapsed": "0.000395s",
      "lines": "all",
      "oss_lines": "all",
      "matched": "100%",
      "vendor": "scanoss",
      "component": "webhook",
      "version": "1.0",
      "latest": "1.0",
      "url": "https://github.com/scanoss/webhook/archive/1.0.tar.gz",
      "file": "webhook-1.0/scanoss/github.py",
      "size": "6624",
      "dependencies": [],
      "licenses": []
    }
  ]
}
$
```

The following example adds a LF to the end of a file. The modified file does not match entirely anymore, but instead the snippet detection comes into effect:

```
$ echo -e "\n" >> webhook-1.0/scanoss/github.py
$ scanoss webhook-1.0/scanoss/github.py
{
  "webhook-1.0/scanoss/github.py": [
    {
      "id": "snippet",
      "elapsed": "0.001812s",
      "lines": "1-192",
      "oss_lines": "3-194",
      "matched": "99%",
      "vendor": "scanoss",
      "component": "webhook",
      "version": "1.0",
      "latest": "1.0",
      "url": "https://github.com/scanoss/webhook/archive/1.0.tar.gz",
      "file": "webhook-1.0/scanoss/github.py",
      "size": "6624",
      "dependencies": [],
      "licenses": []
    }
  ]
}
$
```

