# Working with Persistent Identifiers - Hands-on
This lecture illustrates the use of PIDs, more specifically it shows how to employ [handles](handle.net) using the [B2HANDLE library](https://github.com/EUDAT-B2SAFE/B2HANDLE).

## Prerequisites
If you are not working on one of our test machines you need to install the B2HANDLE library and apply for a prefix. For instructions please follow the documentation:

https://github.com/EUDAT-B2SAFE/B2HANDLE/blob/master/README.md

http://eudat-b2safe.github.io/B2HANDLE/handleclient.html#authentication

## Warming-up: Using PIDs
Below  you find three different PIDs and their corresponding global resolver

- Handle 

    PID: 11304/cf8956a2-39d3-11e5-8a18-f31aa6f4d448

    Resolver: http://hdl.handle.net/

- Doi

    PID: 10.3389/fgene.2013.00289

    Resolver: http://dx.doi.org/

- Ark 

    PID: ark:/13030/tf5p30086k

    Resolver: https://nbn-resolving.org/

You can either go to the resolver in your webbrowser and type in the PID to get to the data behind it. You can also concatenate the resolver and the PID.

**Try to resolve the handle PID with the DOI resolver and vice-versa.**

**In the handle resolver you will find a box "Don't redirect to URLs", if you tick this box, what information do you get?**

Each PID consists of a *prefix* which is linked to an administratory domain (e.g. a journal) and a *suffix*. The prefix is handed out by an issuer such as CNRI for handle or DataCite for DOIs. Once you are admin of a prefix, you can register as many data objects as you want by extending the prefix with a suffix. Note, that the suffixes need to be unique for each data object. The epic client helps you with that.


## Managing PIDs

#### Training machine
On our user interface machines we already installed all necessary packages and the B2HANDLE library. You will find your credentaials for this tutorials in the folder */home/<user>/credentials/cred_b2handle*. 

#### Own laptop
In case you are working on your own laptop with your own python, please install the B2HANDLE library.

## Managing PIDs 
How do repositories create PIDs for data objects?
How can you create a PID for your own data objects?

#### Example workflow
1. Obtain a prefix from an resolver admin (See Prerequisites)
2. Set up internet connection to the PID server with a client
3. Create a PID
4. Link PID and location of the data object

In the tutorial below we will work with a test handle server located at SURFsara. That means the PIDs we create are not resolvable via the global handle resolver or via the DOI resolver.

#### For resolving PIDs please use:
`http://epic3.storage.surfsara.nl:8001`

### Import necessary libraries:

```py
from b2handle.clientcredentials import PIDClientCredentials
from b2handle.handleclient import EUDATHandleClient

import uuid
import hashlib
import os, shutil
```
### Connect to the SURFsara handle server 
To connect to the epic server you need to provide a prefix, the private key and the certificate; alternatively the library also provides authentication with username/password. This information is stored in a json file *cred_file.json* and should look like this:
```sh
{
    "handle_server_url": "https://epic3.storage.surfsara.nl:8001",
    "private_key": "/<full path>/credentials/cred_b2handle/355_841_privkey.pem",
    "certificate_only": "/<full path>/credentials/cred_b2handle/355_841_certificate_only.pem",
    "prefix": "841",
    "handleowner": "200:0.NA/841",
    "reverse_username": "841",
    "reverse_password": "****",
    "HTTPS_verify": "True"
}
```
On the user interface machines you can find such a file and all necessary certificates and keys in */<full path>/credentials/cred_b2handle/*. Please adopt the *<full path>* appropriately.

- Parse credentials
```py
cred = PIDClientCredentials.load_from_JSON('<full_path>/cred_file.json')
```
- Retrieve some information about the server, this server also hosts the resolver which we will use later
```py
print('PID prefix ' + cred.get_prefix())
print('Server ' + cred.get_server_URL())
```

- Create an instance of the client by oassing your credentials:
```py
ec = EUDATHandleClient.instantiate_with_credentials(cred)
```

## Registering a file
### We will register a public file from figshare. 
First store the file location.
```py
fileLocation = 'https://ndownloader.figshare.com/files/2292172'
```

### Building the PID:
- Create a universally unique identifier (uuid)
- Take function for this from
```py
import uuid
uid = uuid.uuid1()
print(uid)
print(type(uid))
```

- Concatenate your PID prefix and the uuid to create the full PID
```py
pid = cred.get_prefix() + '/' + str(uid)
print(pid)
```

We now have an opaque string which is unique to our resolver since
the prefix is unique (handed out by administrators of the resolver).
The suffix has been created with the uuid function. 

- Link the PID and the data object. We would like the PID to point to the location we stored in *fileLocation*

```py
Handle = ec.register_handle(pid, fileLocation)
```

Let’s go to the resolver and see what is stored there
`http://epic3.storage.surfsara.nl:8001`. 
We can get some information on the data from the resolver.
We can retrieve the data object itself via the web-browser.

**Download the file via the resolver. Try to use *wget* when working remotely on our training machine.**

**How is the data stored when downloading via the browser and how via *wget*?**

**Have a look at the metadata stored in the PID entry.**

**What happens if you try to reregister the file with the same PID?**
```py
newHandle = ec.createHandle(pid, fileLocation)
```

### Store some handy information with your file
- We can store some more information in the PID entry with the function *modifyHandle*
```py
?ec.modify_handle_value
```
We can update and create several key-value pairs in one go. To this end we create a python dictionary and pass it to the function.
```py
args = dict([('TYPE', 'file')])
ec.modify_handle_value(Handle, ttl=None, add_if_not_exist=True, **args)
```

- We want to store information on identity of the file, e.g. the md5 checksum. We first have 
to generate the checksum. However, we can only create checksums for files which we 
have access to with our python compiler. In the step above we can download the file and
then continue to calculate the checksum. **NOTE** the filename might depend on the download method.

```py
import hashlib
md5sum = hashlib.md5('/<PATH>/surveys.csv').hexdigest()
args = dict([('TYPE', 'file'), ('MD5', md5sum)])
ec.modify_handle_value(Handle, ttl=None, add_if_not_exist=True, **args)
```

- With the resolver we can access this information. Note, this data is publicly available to anyone.

- We can also access the information via the client:
    ```py
    ec.retrieve_handle_record(Handle)
    ```

### Updating PID entries
- Assume location of file has changed. This means we need to modify the URL field.

```py
ec.modify_handle_value(Handle, ttl=None, add_if_not_exist=True, **dict([('URL', '/<PATH>/surveys.csv')]))
```

**Try to fetch some metadata on the file from the resolver.**

**Try to resolve directly to the file. What happens?**

We updated the "URL" with a local path on a personal machine. That means you can no longer download the data
directly, but you have access to the data stored in the PID.

* Information stored with the PID is ALWAYS public
* Data itself can lie on a protected server/computer and not be accessible for everyone

## Linking two files
We know that the file in the figshare repository and our local file are identical. We want to store this information
in the PIDs.

- Reregister the figshare file
- First create a new PID
```py
uid = uuid.uuid1()
print(uid)
pid = cred.get_prefix() + '/' + str(uid)
```

- Link the new PID/handle to the public figshare data which is still stored in *fileLocation*

```py
newHandle = ec.register_handle(pid, fileLocation)
```

- Leave information that local file should be the same as the figshare file

```py
ec.modify_handle_value(Handle, ttl=None, add_if_not_exist=True, **dict([('REPLICA', newHandle)]))
```

### Reverse look-ups
**TODO**
The epic API extends the handle API with reverse look-ups. Assume you just know some of the metadata stored with a PID but not the full PID. How can you get to the URL field to retrieve the data?

We can fetch the first data with a certain checksum:
```py
args = dict([('CHECKSUM', str(''.join(md5sum)))])
Handle = ec.search_handle(**args)
url = ec.get_value_from_handle(Handle, 'URL')
print(url) 
```

### Using the epicclient Command Line Interface (CLI)
For now we directly worked with the library. EUDAT provides an [epicclient](https://github.com/EUDAT-B2SAFE/B2SAFE-core/blob/master/cmd/epicclient2.py) which can be used as command line interface (CLI) based on the B2HANDLE. 
You can list all options for the CLI on the commandline with:

```sh 
python epicclient2.py os <full path>/cred_file.json -h
```

The functions are adjusted to the functionality in the EUDAT B2SAFE service, but can serve as reference implementation for other use cases.
