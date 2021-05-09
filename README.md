# Bash CLI to manage SPXP profiles

A simple shell script to setup and manage SPXP profiles from your local
machine. Profiles can be published via any service provider supporting
the SPXP-PME and SPXP-SPE protocol extensions -- like https://spxp.space

### Prerequisites
You need a shell like `bash` or `zsh` and
1. [jq](https://stedolan.github.io/jq/) on your path
2. The [SpxpCryptoTool](https://github.com/spxp/spxp-crypto/releases)
   installed and available on your path

### Usage
This tool will setup all cryptographic material that is associated with a
profile for you and store it in your home folder in `~/.spxp/default`. If needed,
you can maintain multiple different identities.

To get started, you should first check if this script can be executed and can
access all its prerequisites. Please run:
```
$ spxp
```
This will validate your system and print out a command list.

#### Setting up a new profile
Simply run:
```
$ spxp init --name "John Doe" --shortInfo "Exploring SPXP..."
```
This will create new keypairs and an initial profile in `~/.spxp/default`.

To publish your newly created profile, you need to bind it to a service provider
like https://spxp.space. To check if a service provider supprts the SPXP-SPE
extension, simply run:
```
$ spxp discover spxp.space
```
This will discover the service provider and print out important API endpoint. For
spxp.space, you will see:
```
Start: https://spxp.space/.spxp-spe/register
Bind: https://spxp.space/.spxp-spe/bind
ManagementEndpoint: https://spxp.space/manage/v01
```
Now you need to use the start URI in a web browser to register with this service
provider. You need to attach `?return_scheme=abcd` to the URI. For example for
spxp.space, this will give you  
https://spxp.space/.spxp-spe/register?return_scheme=abcd  
Open the above link in a new tab and then follow the instruction on this page.
You need to pick a new "slug", under which your profile will be published. For
example, you could chose  
https://spxp.space/john.doe  
After finishing the registration process, right-click the "Continue with profile
setup" link and chose "Copy Link Address".  
Then paste the copied link into this command:
```
$ spxp bind spxp.space nSMGCJeraEzOfa7p
```
You need to remove the `abcd:` prfix from the link you copied. This will then
bind you local profile to the slug you have chosen on the website and publish
your initial profile.  
The command will print out your profile URI like this:
```
Successfully bound identity 'default' to 'https://spxp.space/john.doe'. To validate, use:
curl https://spxp.space/john.doe
```
You can now curl your profile URI or open this URI in a web browser to see your
published profile.

#### Updating your profile
To update the information on your profile, like your "About" section, simply run:
```
$ spxp profile about "People say nothing is impossible, but I do nothing every day."
```
This will update your profile in `~/.spxp/default/profile.json`, sign it with
your profile key, and then publish it. To remove a value, simply run:
```
$ spxp profile about
```

#### Posting
To publish a new post, simply run
```
$ spxp post text --message "Hello, world\!"
```
Please make sure to encode special characters properly dending on your shell.  
To post a new photo, you need to provide the path to a local JPEG or PNG image
like this:
```
$ spxp post photo --message "Look at this" --preview /path/to/local/image.jpeg
```
Preview images should be of about 250KB and within 512 to 512 pixels. There are
no hard limits, but clients might suppress posts if they exceed certain limits.

#### Multiple identities
You can manage multiple identities independently on your machine. Simply append
```
--identity <name>
```
to any command. This will maintain the related files in `~/.spxp/<name>`.  
For example:
```
$ spxp init --name "Jane Doe" --shortInfo "Hey there..." --identity jane
$ spxp bind spxp.space nSMGCJeraEzOfa7p --identity jane
$ spxp profile about "Social media geek." --identity jane
$ spxp post text --message "Hello, world\!" --identity jane
```