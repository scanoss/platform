# SCANOSS Platform Deployment Guide

## Overview

In order to have access the full capabilities of SCANOSS you need to deploy a number of components:
- The [**LDB**](https://github.com/scanoss/ldb), the database that will contain the Knowledge Base
- The [**engine**](https://github.com/scanoss/engine), the inventory engine, that provides scanning and searching capabilities over LDB
- The [**API**](https://github.com/scanoss/API), provides a simple RESTful API for SCANOSS that can be accessed remotely by any clients, such as [scanner.py](https://github.com/scanoss/scanner.py)

These 3 components will give you the capability of scanning and performing Software Composition Analysis on your source code. However, your *Knowledge Base* (LDB) is empty, and you will need to import some software components into it to be able to see some interesting results.

In order to maintain **LDB**, your knowledge base, you will need to install **minr** and implement a [Mining process](MINING.md) to import data into your instance of **LDB**.

## Hardware Requirements

The SCANOSS Platform requires at least a Intel(R) Xeon(R) E-2236 CPU or equivalent with 32Gb of RAM or more. Diskspace requirements for an entire production Knowledgebase is 10Tb. 

## Software Requirements

The SCANOSS Platform runs on Linux, but it could certainly be ported to Windows or OSX as it is written entirely in C. This guide is written for Ubuntu Server 20.04 LTS. Package names may vary depending on the Linux distribution used. 

Before installing, make sure all software requirements are in place:

```
sudo apt install build-essential libssl-dev zlib1g-dev libsqlite3-dev libz-dev curl gem ruby unzip p7zip-full unrar-free
```

And that's it! SCANOSS only requires a number of standard libraries that are pretty much already present in any Linux distribution. 

From here, the installation process for each component is pretty much the same:
1. Checkout component project from GitHub
2. Run `make` and additionally `make install` of the component
3. Use it!

## Installation from sources

### LDB Installation

LDB is the database engine used for the Knowledge Base. The LDB is required for the engine and minr. The source code can be downloaded and compiled as follows:

```
wget -O ldb.zip https://github.com/scanoss/ldb/archive/master.zip
unzip ldb.zip
cd ldb-master
make
make lib
sudo make install
sudo mkdir -p /var/lib/ldb/oss/{purl,url,file,wfp}
sudo chown -R user.user /var/lib/ldb   # Replace user.user with your user and group
cd ..
ldb
```

The ldb prompt shows the LDB has been properly installed. You can simply quit as follows: 

```
Welcome to LDB 2.02
Use help for a command list and quit for leaving this session

ldb> quit
```

### Engine Installation

See [Installation Section](https://github.com/scanoss/engine#installation) on the engine's `README`.

### API Installation

See [Installation Section](https://github.com/scanoss/API#installation) on the API's `README`.

### Installing minr

Minr is the command-line tool used for downloading and indexing source code. The source code can be downloaded and compiled as follows:

```
wget -O minr.zip https://github.com/scanoss/minr/archive/master.zip
unzip minr.zip
cd minr-master
make all
sudo make install
cd ..
minr -v
```

The last command should show the installed version of minr.

### Installing the Inventory Engine

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

## Testing minr and the Inventory Engine

The following examples show the entire process for downloading an OSS component, importing it into the Knowledge Base and performing a scan against it using the SCANOSS Open Source Inventory Engine:


### URL mining

URL mining is the process of downloading a component, expanding the files and saving component, metadata and original sources for snippet mining.

```
$ minr -d scanoss,webhook,1.0,20200320,BSD-3-Clause,pkg:github/scanoss/webhook@1.0 -u https://github.com/scanoss/webhook/archive/1.0.tar.gz
Downloading https://github.com/scanoss/webhook/archive/1.0.tar.gz
$
```

A mined/ directory is created with the downloaded metadata. This includes component and file metadata and source code archives which are kept using the .mz archives, specifically designed for this purpose.

### Snippet mining

Snippet mining is a second step, where snippet information is mined from the .mz archives. This is achieved with the -z parameter.

```
$ minr -z mined
mined/sources/146a.mz
...
$
```

A mined/snippets directory is created with the snippet wfp fingerprints extracted from the .mz files located in mined/sources

### Data importation into the LDB

```
$ minr -i mined/
$
```

The LDB is now loaded with the component information and a scan can be performed.

### Scanning against the LDB Knowledge Base

The following example shows an entire component match:

```
$ scanoss 1.0.tar.gz
{
  "1.0.tar.gz": [
    {
      "id": "file",
      "status": "pending",
      "lines": "all",
      "oss_lines": "all",
      "matched": "100%",
      "purl": [
        "pkg:github/scanoss/webhook@1.0"
      ],
      "vendor": "scanoss",
      "component": "webhook",
      "version": "1.0",
      "latest": "1.0",
      "url": "https://github.com/scanoss/webhook@1.0",
      "release_date": "20200320",
      "file": "1.0.tar.gz",
      "url_hash": "611e5c3a58a3c2b78385556368c5230e",
      "file_hash": "611e5c3a58a3c2b78385556368c5230e",
      "file_url": "https://github.com/scanoss/webhook/archive/1.0.tar.gz",
      "dependencies": [],
      "licenses": [
        {
          "name": "BSD-3-Clause",
          "obligations": "https://www.osadl.org/fileadmin/checklists/unreflicenses/BSD-3-Clause.txt",
          "copyleft": "no",
          "patent_hints": "no",
          "source": "component_declared"
        }
      ],
      "copyrights": [
        {
          "name": "Copyright (C) 2017-2020; SCANOSS Ltd. All rights reserved.",
          "source": "license_file"
        }
      ],
      "vulnerabilities": [],
      "quality": [],
      "cryptography": [],
      "server": {
        "hostname": "localhost",
        "version": "4.2.4",
        "flags": "0",
        "elapsed": "0.041169s"
      }
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
      "status": "pending",
      "lines": "all",
      "oss_lines": "all",
      "matched": "100%",
      "purl": [
        "pkg:github/scanoss/webhook@1.0"
      ],
      "vendor": "scanoss",
      "component": "webhook",
      "version": "1.0",
      "latest": "1.0",
      "url": "https://github.com/scanoss/webhook@1.0",
      "release_date": "20200320",
      "file": "webhook-1.0/scanoss/github.py",
      "url_hash": "611e5c3a58a3c2b78385556368c5230e",
      "file_hash": "8c2fa3f24a09137f9bb3860fa21c677e",
      "file_url": "https://osskb.org/api/file_contents/8c2fa3f24a09137f9bb3860fa21c677e",
      "dependencies": [],
      "licenses": [
        {
          "name": "BSD-3-Clause",
          "obligations": "https://www.osadl.org/fileadmin/checklists/unreflicenses/BSD-3-Clause.txt",
          "copyleft": "no",
          "patent_hints": "no",
          "source": "component_declared"
        }
      ],
      "copyrights": [
        {
          "name": "Copyright (C) 2017-2020; SCANOSS Ltd. All rights reserved.",
          "source": "license_file"
        }
      ],
      "vulnerabilities": [],
      "quality": [],
      "cryptography": [],
      "server": {
        "hostname": "localhost",
        "version": "4.2.4",
        "flags": "0",
        "elapsed": "0.129558s"
      }
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
      "status": "pending",
      "lines": "1-192",
      "oss_lines": "3-194",
      "matched": "99%",
      "purl": [
        "pkg:github/scanoss/webhook@1.0"
      ],
      "vendor": "scanoss",
      "component": "webhook",
      "version": "1.0",
      "latest": "1.0",
      "url": "https://github.com/scanoss/webhook@1.0",
      "release_date": "20200320",
      "file": "webhook-1.0/scanoss/github.py",
      "url_hash": "611e5c3a58a3c2b78385556368c5230e",
      "file_hash": "8c2fa3f24a09137f9bb3860fa21c677e",
      "file_url": "https://osskb.org/api/file_contents/8c2fa3f24a09137f9bb3860fa21c677e",
      "dependencies": [],
      "licenses": [
        {
          "name": "BSD-3-Clause",
          "obligations": "https://www.osadl.org/fileadmin/checklists/unreflicenses/BSD-3-Clause.txt",
          "copyleft": "no",
          "patent_hints": "no",
          "source": "component_declared"
        }
      ],
      "copyrights": [
        {
          "name": "Copyright (C) 2017-2020; SCANOSS Ltd. All rights reserved.",
          "source": "license_file"
        }
      ],
      "vulnerabilities": [],
      "quality": [],
      "cryptography": [],
      "server": {
        "hostname": "localhost",
        "version": "4.2.4",
        "flags": "0",
        "elapsed": "0.067090s"
      }
    }
  ]
}

$
```

