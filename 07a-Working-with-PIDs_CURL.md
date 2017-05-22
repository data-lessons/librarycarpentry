# Working with Persistent Identifiers - Hands-on
This lecture takes you through the steps to create and administer PIDs employing the HTTP restful API.


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

You can either go to the resolver in your web browser and type in the PID to get to the data behind it. You can also concatenate the resolver and the PID.

**Try to resolve the handle PID with the DOI resolver and vice versa.**

**In the handle resolver you will find a box "Don't redirect to URLs", if you tick this box, what information do you get?**

Each PID consists of a *prefix* which is linked to an administratory domain (e.g. a journal) and a *suffix*. The prefix is handed out by an issuer such as CNRI for handle or DataCite for DOIs. Once you are admin of a prefix, you can register as many data objects as you want by extending the prefix with a suffix. Note, that the suffixes need to be unique for each data object. The epic client helps you with that.

## Managing PIDs

### Prerequisites

The code is based on cURL. cURL is an open source command line tool and library for transferring data with URL syntax. cURL is used in command lines or scripts to transfer data.

 * Please check the dependencies before you start.
 * You will also need test credentials for the epic server.


#### Install cURL dependencies

CURL: is an open source command line tool and library for transferring data with URL syntax.
On the training machines 

```py
apt-get install curl 
apt-get install uuid-runtime 
```

#### Own laptop

In case you are working on your own laptop with your own python, please install:

```py
easy_install curl
easy_install uuid-runtime
```

Final check

```py
curl --help
```

If you write the code described below in a file, don't forget to change the permissions. 
You should make each file executable. 

Suppose you have a file `filename.sh` 

you can make it by doing as:
`chmod +x filename.sh`

so it will execute when you type
`./filename.sh`


#### Example workflow

1. Obtain a prefix from an resolver admin
2. Set up internet connection to the PID server with a client
3. Create a PID
4. Link PID and location of the data object

In the tutorial below we will work with a test handle server located at SURFsara. 
That means the PIDs we create are not resolvable via the global handle resolver or via the DOI resolver.

#### For resolving PIDs please use:

`http://epic3.storage.surfsara.nl:8001`

### Main Parameters of CURL 

The main command
 
  `curl [options] [URL...]`
  
  
Before we start, lets explain the main parameters of CURL used as options 

 * **-X, --request <command>**: (HTTP) Specifies a custom request method to use when communicating with the HTTP server. The specified request method will be used instead of the method otherwise used (which defaults to GET).  Common additional HTTP requests include PUT ,POST and DELETE . ( -X GET) 
 * **-U, --proxy-user <user:password>**: Specify the user name and password to use for proxy authentication. (ex: -u "username:pass) 
 * **-H, --header <header>**: Extra header to include in the request when sending HTTP to a server. You may specify any number of extra headers.(ex: -H "Accept: application/json" so as to accept json data)
 * **-d, --data <data>**: (HTTP) Sends the specified data in a POST or PUT request to the HTTP server, in the same way that a browser does when a user has filled in an HTML form and presses the submit button. 
 * **-D, --dump-header <file>**: Write the protocol headers to the specified file.
 * **-v**: get verbosed information about the connection with the server 
 
These are the main parameters we are going to use in our examples. For more parameters please check [cURL]Chttps://curl.haxx.se/)

### Connect to the SURFsara handle server 

To connect to the epic server you need to provide a prefix and a password. 
If you use the example files, this information is stored in a *config.txt* file  and should look like this:

```py
USERNAME=846
PASSWORD=xxxx
FILENAME=surveys.csv #the file (and its location) we are going to use in the examples
PID_SERVER=https://epic3.storage.surfsara.nl/v2_test/handles/846 #be carefull not to add the trailing slash
PID_SUFFIX=XXXX #the suffix of the first created handle
PID2_SUFFIX=YYYY #the suffix of the second handle
```
You can find the the username and password on the user interface machine in *credentials/cred_epic/cred_file.json*.

 - Get the list of handles in 846 prefix as an example. 
```py
curl -u "846:XXX" -H "Accept: application/json" \
	 -H "Content-Type: application/json" \
	 https://epic3.storage.surfsara.nl/v2_test/handles/846/
```

- Connect with your credentials (username, password)
```py
-u "846:XXX"
```
- Use the correct headers to send and accept json format 

```py
-H "Accept: application/json" -H "Content-Type: application/json"
```

- The PID prefix is combined with the pid server url  

```py
https://epic3.storage.surfsara.nl/v2_test/handles/846/
```

## Registering a file

### We will register a public file from figshare. 

First prepare the data in a json format to register. 

```py
'[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}]'
```

We are going to create a new PID by using the PUT request

So the request method is -X PUT followed by the actual json data 

```py
-X PUT --data '[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}]'
```

### Building the PID:

- Create a universally unique identifier (uuid)
- Take function for this from
```py
SUFFIX=`uuidgen`
```

- Concatenate your PID prefix and the uuid to create the full PID
` 846/$SUFFIX `

We now have an opaque string which is unique to our resolver (846/$SUFFIX ) since
the prefix is unique (handed out by administrators of the resolver).

The URL in the CURL request: 
```py
https://epic3.storage.surfsara.nl/v2_test/handles/846/$SUFFIX 
```

- Register the PID, link the PID and the data object. We would like the PID to point to the location we stored in *fileLocation*
(example1.sh)

```py
#!/bin/bash    

SUFFIX=`uuidgen`

curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X PUT --data '[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}]'\
		 https://epic3.storage.surfsara.nl/v2_test/handles/846/$SUFFIX 
```

The result of this request is a new handle with the name HANDLE where
HANDLE = 846/UUID1

#### Responses 
 - HTTP/1.1 201 Created: (Success)
 - HTTP/1.1 204 No-Content: The local name already exists , and instead of creating a new one you’ve just updated the values of an existing one.
 - HTTP/1.1 401 Unauthorized: Your username or your password is wrong
 - HTTP/1.1 405 Method Not Allowed: 
 	- You are trying to create a new handle in the main url of the server either (https://epic.grnet.gr/handles/11239/) or (https://epic.grnet.gr/handles). You have not specified a unique name for your handle. (or)
	- You are trying to create a new handle with manual generation of suffix name via POST instead of PUT. POST supports automatic generation of suffix name.
 - HTTP/1.1 415 Unsupported Media Type: You haven’t specify the correct headers for your request. The service supports Json representation so you must define the content-type of the request.


Let’s go to the resolver and see what is stored there
`http://epic3.storage.surfsara.nl:8001`. 
We can get some information on the data from the resolver.
We can retrieve the data object itself via the web-browser.

**Download the file via the resolver. Try to use *wget* when working remotely on our training machine.**

**How is the data stored when downloading via the browser and how via *wget*?**

**Have a look at the metadata stored in the PID entry.**

**What happens if you try to reregister the file with the same PID?** 

Dont forget to change the UUD1 to the correct suffix value.

```py
curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X PUT --data '[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}]'\
		 https://epic3.storage.surfsara.nl/v2_test/handles/846/UUID1 
```
(Use example2.sh)

### Store some handy information with your file

- We can store some more information in the PID entry by modifying the json data 

Lets say that we want to add a new type with data 'Data Carpentry pandas example file'.
We have to update the json data
```
[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}, 
 {"type":"TYPE","parsed_data":"Data Carpentry pandas example file"}]

```

And the actual request is:

```py
curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X PUT --data '[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}, {"type":"TYPE","parsed_data":"Data Carpentry pandas example file"}]'\
		 https://epic3.storage.surfsara.nl/v2_test/handles/846/UUID1
```

Use example3.sh

- We want to store information on identity of the file, e.g. the md5 checksum. We first have 
to generate the checksum. However, we can only create checksums for files which we 
have access to with our python compiler. In the step above we can download the file and
then continue to calculate the checksum. **NOTE** the filename might depend on the download method.

```#!/bin/bash
md5value=` md5 surveys.csv | awk '{ print $4 }'`
curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X POST --data '[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}, 
 					    {"type":"TYPE","parsed_data":"Data Carpentry pandas example file"}, 
 					    {"type":"MD5","parsed_data":$md5value}]'\
		 https://epic3.storage.surfsara.nl/v2_test/handles/846/UUID1

```

Use example4.sh

- With the resolver we can access this information. Note, this data is publicly available to anyone.

### Reverse look-ups
The epic API extends the handle API with recursive look-ups. Assume you just know some of the metadata stored with a PID but not the full PID. How can you get to the URL field to retrieve the data?

We can fetch the first data with a certain checksum:
```py
curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X GET \
		https://epic3.storage.surfsara.nl/v2_test/handles/846/?MD5=MD5VALUE
```

Use example5.sh

#### Responses 
 - HTTP/1.1 200 OK: (Success)
 - HTTP/1.1 401 Unauthorized: Your username or your password is wrong
 - HTTP/1.1 404 NOT found: The url doesn’t exist
 
### Updating PID entries
- Assume location of file has changed. This means we need to modify the URL field.

```py
curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X POST --data '[{"type":"URL","parsed_data":"/<PATH>/surveys.csv"}]'\
		 https://epic3.storage.surfsara.nl/v2_test/handles/846/UUID1
```

Use example6.sh

#### Responses
 - HTTP/1.1 204 No-Content: The local name already exists , and instead of creating a new one you’ve just updated the values of an existing one.
 - HTTP/1.1 401 Unauthorized: Your username or your password is wrong
 - HTTP/1.1 415 Unsupported Media Type: You haven’t specify the correct headers for your request. The service supports Json representation so you must define the content-type of the request.
 
**Try to fetch some metadata on the file from the resolver.**

**Try to resolve directly to the file. What happens?**

We updated the "URL" with a local path on a personal machine. That means you can no longer download the data
directly, but you have access to the data stored in the PID.

* Information stored with the PID is ALWAYS public
* Data itself can lie on a protected server/computer and not be accessible for everyone

### Linking two files

We know that the file in the figshare repository and our local file are identical. We want to store this information
in the PIDs.

- Reregister the figshare file
- First create a new PID
```py
#!/bin/bash    

SUFFIX=`uuidgen`

curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X PUT --data '[{"type":"URL","parsed_data":"https://ndownloader.figshare.com/files/2292172"}]'\
		 https://epic3.storage.surfsara.nl/v2_test/handles/846/$SUFFIX 
```

- Leave information that local file should be the same as the figshare file. Update the data of handle 

First update the json 
```
	[{"type":"URL","parsed_data":"/<PATH>/surveys.csv"},{"type":"SAME_AS","parsed_data":"846/newhandle"}]
```

```py
curl -v -u "YOURUSERNAME:YOURPASSWORD" -H "Accept:application/json" \
		-H "Content-Type:application/json" \
		-X POST --data '[{"type":"URL","parsed_data":"/<PATH>/surveys.csv",},{"type":"SAME_AS","parsed_data":"846/newhandle"}]'\
		 https://epic3.storage.surfsara.nl/v2_test/handles/846/UUID1
```




These examples are adjusted to the functionality in the EUDAT B2SAFE service, but can serve as reference implementation for other use cases.

