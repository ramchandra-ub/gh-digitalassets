# gh-digitalassets
A gh cli extension which can be used to perform CRUDS(Create, read, update, delete, search) and other equivalent operations on the digital assets right from the command line.

## Motivation
In this rapidly evolving digital world, every organization, be it an educational institution, a logistics company, or any other company, possesses an extensive database of digital assets. However, keeping track of these assets has become challenging. Organizations often need to collaborate with others to share these assets, but the process can be cumbersome, with difficulties in sharing, monitoring changes, and searching through vast databases. To simplify this problem, we have collaborated with IBM to propose a solution that streamlines the organization of digital assets and facilitates their sharing among different organizations. Our solution aims to enhance asset management and simplify the process of asset sharing between organizations. Through the use of self-describing metadata and categorized tags, we enable efficient searching, streamlined collaboration, and improved visibility into the digital assets owned by each organization.


To facilitate this, we have created a cli tool which is an extension on top of the Github CLI.


## Installation
First, the GitHub Cli must be installed. Please follow this link https://github.com/cli/cli#installation to install GitHub Cli.

Next, the extension needs to be installed.
```console
gh extension install https://github.com/ramchandra-ub/gh-digitalassets
```

### Prerequisites
The following dependencies needs to be installed before using the extension.
- argparse
- github
- InquirerPy
- base64
- requests
- fuzzywuzzy

Please use the below code to install the dependencies
```console
pip install argparse github InquirerPy requests fuzzywuzzy

```

## Usage
This project is mainly created to make the digital asset sharing and collaboration easier. It supports CRUDS(Create, read, update, delete, search) and other equivalent operations on the digital assets from the command line. Digital assets can be anything like software code, datasets, ML models etc.

Running ```gh digitalassets --help``` will show all the commands that are available and what they do
```console
usage: digitalassets [-h]
                     {updateFile,subscribeRepo,getStatus,createRepo,downloadRepo,downloadFile,listContents,readMetadata,deleteFile,search,list}
                     ...

Simple helper commands to interact with Github Repos

positional arguments:
  {updateFile,subscribeRepo,getStatus,createRepo,downloadRepo,downloadFile,listContents,readMetadata,deleteFile,search,list}
                        Below are the commands that are supported with their descriptions
    updateFile          Update your digital asset files
    subscribeRepo       Subscribe to a repo to get updates on the repo
    getStatus           Get the status of the repo
    createRepo          Create a digital asset repo on GitHub
    downloadRepo        Provide the name of the repo to download it as Zip or Tar file
    downloadFile        Download an individual file from the repo. Requires repo name and file
                        name
    listContents        List the contents of the repo
    readMetadata        Read the metadata file from the repo
    deleteFile          Delete a file from the repo. Requires repo name and file name
    search              Search for digital assets using topics, name or keywords
    list                List all the digital assets

options:
  -h, --help            show this help message and exit
```

### Create
Creates a repo on GitHub by uploading the files specified. It is required for the repo to have data files and a supporting metadata file which shows the type of data that is being uploaded. Please find the metadata format below. It must be a json file of name metadata.json.
```json
{
    "assetName": "Linear Regression",
    "description": "This is a linear regression model",
    "topics": [
        "ens-machinelearning",
        "ens-asset"
    ],
    "associations": [
        "www.google.com",
        "www.facebook.com"
    ]
}
```
- assetName: It will be the digital asset name which is the same as repo name.
- description: A one line description about the type of digital asset that is being uploaded.
- topics: list of topics to identify the type of the digital asset
- associations: URLs of repos that are similar to the current digital asset. In future the tool will automatically populate this field with related URLs.

#### Constraints on the metadata file
It must have all the keys as shown above and there must be one topic from the list below. If any of this is missing the tool will be able to interactively create it as we will see below.
```
❯ ens-machinelearning
  ens-mathematics
  ens-lifesciences
  ens-sports
  ens-others
  ```

Running ```gh digitalassets createRepo --help``` will give the below help on create.

```console
usage: digitalassets createRepo [-h] --reponame REPONAME [--path PATH] [--empty EMPTY]

options:
  -h, --help           show this help message and exit
  --reponame REPONAME  Give the name of the repo to be created
  --path PATH          Give a folder path to upload files. If not given will take the current
                       directory
  --empty EMPTY        True if we need to create empty repo. By defualt False
```

Here reponame is a mandatory parameter. path is not mandatory. It will upload all files in the current directory if path is not specified. user will be given a warning like below before proceeding with the current directory.

```console
>gh digitalassets createRepo --reponame testrepo
Files must be uploaded when creating a non-empty repo
Files in the current repo are:
['metadata.json', 'test.py']
Do you want all files in the current directory to be uploaded?  Press Y/N: Pressing N will not create any repo.
```
Pressing N will not create any repo and changes can be made.

#### Sample create repo flow when path is given and the metadata.json has all the required fields.
```console
>gh digitalassets createRepo --reponame testrepo3may1 --path C:\Users\ramnu\UploadDigitalAsset
Repo created with name ramchandra-ub/testrepo3may1
['metadata.json', 'test.py']
All files in the specified directory uploaded
✓ Edited repository ramchandra-ub/testrepo3may1
```
It will create a repo and upload the files in the repo. It will also crawl through the metadata file and update the GitHub topics and description with the relevenat content.

#### Sample create repo flow when path is given and the metadata.json is not found
The tool will be able to interactively create the metdata file as shown below.
```console
>gh digitalassets createRepo --reponame testrepo3may2 --path C:\Users\ramnu\UploadDigitalAsset
Metadata file not found. It is mandatory for a digital asset.
You can create it interactively. Press Y to create interactively, or press N to create it manually: Y
Please provide the asset Name.
TestAsset
Please provide a one liner description
This is a test description
Please select a mandatory topic from below
? What topic to add?
❯ ens-machinelearning
  ens-mathematics
  ens-lifeSciences
  ens-Sports
  ens-others
#### User can select one mandatory topic using the keyboard
? What topic to add? ens-machinelearning
You can also add custom topics to this digital asset. Press Y if you want to add new topics to the repo. Otherwise press N: Y
Enter the topic names using comma separation: test1,test2
Please provide the associations(URSL's). If multiple, provide by giving space
www.google.com www.facebook.com
Repo created with name ramchandra-ub/testrepo3may2
['metadata.json', 'test.py']
All files in the specified directory uploaded
```

When the metadata.json file is present, but a few fields are missing the tool will be able to interactively create those missing fields and upload it to the repo and change the file locally.

### Search

Search for digital assets using name, topics and keywords

#### Search by name

Search by giving the name of the digital asset which will be same as the name of the repo

command 
```
>gh digitalassets search --name reponame
```
example for search by name 
```
>gh digital_asset search --name test
1
Name: Testrepo8may2 
Link: https://github.com/ramchandra-ub/Testrepo8may2 
Uploaded By: ramchandra-ub

2
Name: testrepo12may1-18 
Link: https://github.com/ramchandra-ub/testrepo12may1-18 
Uploaded By: ramchandra-ub

```

#### Search by topics

Search by giving the topic associated with the digital asset, multiple topics can be given seperated by comma
```
>gh digitalassets search topics topic1,topic2

```
example search by topics
```
>gh digitalassets search --topics sample-topic
1
Name: testrepo12may1-4 
Link: https://github.com/ramchandra-ub/testrepo12may1-4 
Uploaded By: ramchandra-ub

2
Name: testrepo12may1-3 
Link: https://github.com/ramchandra-ub/testrepo12may1-3 
Uploaded By: ramchandra-ub
```
#### Search by keyword

Searches the description of the asset, multiple keywords can be given seperated by comma
```
gh digitalassets search --keyword keyword1,keyword2

```
example for search by keywords
```
>gh digitalassets search --keywords test,sample 
1
Name: test-digital-asset 
Link: https://github.com/Harichandana-Vejendla/test-digital-asset 
Uploaded By: Harichandana-Vejendla

2
Name: sample-digital-asset 
Link: https://github.com/Harichandana-Vejendla/sample-digital-asset 
Uploaded By: Harichandana-Vejendla
```

### Update

Update the data/meta-data file or both the files at once. 
```console
-h, --help            show this help message and exit
  --reponame REPONAME   Give the name of the repo to be updated
  --updateType UPDATETYPE
                        Mention the type of the file to be updated i.e. meta-data or data
  --updateFile UPDATEFILE
                        Mention the name of the file which you want to update
  --filepath FILEPATH   Path of the directiry containing your metadata and data parts
```

update syntax
```
gh digitalasset updateFile --reponame <reponame> --updateType <metadata> --updateFile <filename> --filepath <path>
```

The reponame, updateType parameters are mandatory. updateType takes 3 parametes.
1. data - it only updates the data files
2. metadata - it only updates the metadata file
3. both - it updates both data and metadata file.
updateFile, filepath parameters are optional. 

updateFile is the name of the file in the repo which you want to update and if the filepath is given, then the the tool will look for the meta-data/data files from that particular directory. Else it will look in the current directory.
If the meta-data file is to be updated and there is no file or if the file is not according to the standards of the digital asset, then the tool will handle it  interactively as shown above in the create part.

### Subscribe

Subscribe to a particular asset in your community after you search/download it.
```
gh digitalasset subscribeRepo --reponame <reponame>
```

It takes a mandatory argument reponame. You subscribe to that asset.

### Get Status

Get the status of an asset to which you have subscribed to.
```
gh digitalasset getStatus --reponame <reponame>
```

It takes a mandatory argument reponame. You subscribe to that asset.

### List 

list all the digital assets

```
gh digitalasset list
```

### Download Repo

Downloads the entire repo either as a Zip file or a Tar file. 

```
gh digitalassets downloadRepo --reponame <reponame>
```

It takes a mandotary argument reponame. It gives option to download the repo either as Zip or Tar file. 

```console
? How would you like to download the repo? 
❯ Zip
  Tar
  
Download Repo Successful
```

### Download File

It provides user the option to download an individual file from the repo. It gives user flexibility to download any file form the repo which is an added feature on top of Github.

```
gh digitalassets downloadFile --reponame <reponame> --filename <filename>
```
It takes mandatory argument repo name and optional argument filename. If file name is not specified then it gives user the flexibility to choose from the list of files available in the repo. It accepts fuzzy search. If the user is not sure of the name of the file then it implements a fuzzy search to odwnload the nearest match of the file. User has the option to specify the extension then it downloads all the files with the particular extension. 

```
gh digitalassets downloadFile --reponame <reponame> --filename <ExtensionName>
```

```console
optional arguments:
  -h, --help           show this help message and exit
  --fileName FILENAME  File name to be downloaded
  --reponame REPONAME  Name of the repo
  ```
  
  Example of a sample file download without any filename specified
  
  ```
  ? Select a file to download 
❯ Blank diagram (1).pdf
  Home.html
  Insertion Testing.ipynb
  metadata.json
  poster.pdf
  NewFolder/Action Log .pdf
  NewFolder/Fibonacci Series.ipynb
  NewFolder/Home.html
  NewFolder/poster.pdf
  ```
  
  ### List Contents
  
  It lists the contents of the repo. It gives back a list of all the files present in the repo.
  
  ```
  gh digitalassets listContents --reponame <reponame>
  ```
  It takes a mandatory argument reponame and lists the content of that repo.
  
  ```console
  optional arguments:
  -h, --help           show this help message and exit
  --reponame REPONAME  Name of the repo
  ```
  
  ### Read Metadata
  
  Allows the user to read the metadata of a particular repo. It displays the contents of the metadata in a readable format.
  
  ```
  gh digitalassets readMetadata --reponame <reponame>
  ```
  
  It takes mandatory argument reponame.
  
  ### Delete File.
  
  It allows the user to delete a particular file from the repo. If the file is not found then it displays an error message that the file is not   found. If the file is present in a folder then the entire path needs to be specified.
  
  ```
  gh digitalassets deleteFile --reponame <reponame> --fileName <fileName>
  
  gh digitalassets deleteFile --reponame <reponame> --fileName <Folder/filename>
  ```
  It takes mandatory arguments reponame and filename.
  
  ```console
  optional arguments:
  -h, --help           show this help message and exit
  --fileName FILENAME  File name to be downloaded
  --reponame REPONAME  Name of the repo
  ```
  















