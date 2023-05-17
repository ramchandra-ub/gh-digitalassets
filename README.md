# gh-digitalassets
A gh cli extension which can be used to perform CRUDS(Create, read, update, delete, search) and other equivalent operations on the digital assets right from the command line.

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
                     {updateFile,createrepo,clone,downloadRepo,downloadFile,listContents,readMetadata,deleteFile,search}
                     ...

Simple helper commands to interact with Github Repos

positional arguments:
  {updateFile,createrepo,clone,downloadRepo,downloadFile,listContents,readMetadata,deleteFile,search}
                        Testing help command
    updateFile          Update your digital asset files
    subscribeRepo       Subscribe to a repo to get updates on the repo
    getStatus           Get the status of the repo
    createrepo          Create a digital asset repo on GitHub
    downloadRepo        Provide the name of the repo to download it as Zip or Tar file
    downloadFile        Download an individual file from the repo. Requires repo name and file name
    listContents        List the contents of the repo
    readMetadata        Read the metadata file from the repo
    deleteFile          Delete a file from the repo. Requires repo name and file name
    search              Search for digital assets using topics, name or keywords

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
  ens-lifeSciences
  ens-Sports
  ens-others
  ```

Running ```gh digitalassets createrepo --help``` will give the below help on create.

```console
usage: digitalassets createrepo [-h] --reponame REPONAME [--path PATH] [--empty EMPTY]

options:
  -h, --help           show this help message and exit
  --reponame REPONAME  Give the name of the repo to be created
  --path PATH          Give a folder path to upload files. If not given will take the current directory
  --empty EMPTY        True if we need to create empty repo. By defualt False
```

Here reponame is a mandatory parameter. path is not mandatory. It will upload all files in the current directory if path is not specified. user will be given a warning like below before proceeding with the current directory.

```console
>gh digitalassets createrepo --reponame testrepo
Files must be uploaded when creating a non-empty repo
Files in the current repo are:
['metadata.json', 'test.py']
Do you want all files in the current directory to be uploaded?  Press Y/N: Pressing N will not create any repo.
```
Pressing N will not create any repo and changes can be made.

#### Sample create repo flow when path is given and the metadata.json has all the required fields.
```console
>gh digitalassets createrepo --reponame testrepo3may1 --path C:\Users\ramnu\UploadDigitalAsset
Repo created with name ramchandra-ub/testrepo3may1
['metadata.json', 'test.py']
All files in the specified directory uploaded
✓ Edited repository ramchandra-ub/testrepo3may1
```
It will create a repo and upload the files in the repo. It will also crawl through the metadata file and update the GitHub topics and description with the relevenat content.

#### Sample create repo flow when path is given and the metadata.json is not found
The tool will be able to interactively create the metdata file as shown below.
```console
>gh digitalassets createrepo --reponame testrepo3may2 --path C:\Users\ramnu\UploadDigitalAsset
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

```
gh digitalassets --search reponame

```
#### Search by topics

Search by giving the topic associated with the digital asset, multiple topics can be given seperated by comma
```
gh digitalassets --search topics topic1,topic2

```

#### Search by keyword

Searches the description of the asset, multiple keywords can be given seperated by comma
```
gh digitalassets --search keyword keyword1,keyword2

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

### Subscribe to repo



















