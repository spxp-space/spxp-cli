# Bash CLI to manage SPXP profiles

A simple shell script to setup and manage SPXP profiles from your local
machine. Profiles can be published via any service provider supporting
the SPXP-PME and SPXP-SPE protocol extensions, like https://spxp.space

### Prerequisites
You need a shell like `bash` or `zsh` and
1. [jq](https://stedolan.github.io/jq/) on your path
2. The [SpxpCryptoTool](https://github.com/spxp/spxp-crypto/releases)
   installed and available on your path

### Install
Download this script and make it available on your path. The location you chose
is up to personal preference. You can put it in `/usr/local/bin` or create a
`~/bin` folder in your home directory and add it to the path. Or any other
place of your liking.

### Usage
This tool will setup all cryptographic material that is associated with a
profile for you and store it in your home folder in `~/.spxp/default`. If needed,
you can maintain multiple different identities on your machine.

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
like https://spxp.space. To check if a service provider supports the SPXP-SPE
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
Now you need to open the start URI in a web browser to register with this service
provider. Attach the query string `?return_scheme=abcd` to the URI. For example for
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
You need to remove the `abcd:` prefix from the link you copied. This will then
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
your profile key, and then publish it.  
To remove a value, simply run this command without a property value:
```
$ spxp profile about
```

#### Posting
To publish a new post, simply run
```
$ spxp post text --message "Hello, world\!"
```
Please make sure to encode special characters of your shell properly.  
To post a new photo, you need to provide the path to a local JPEG or PNG image
like this:
```
$ spxp post photo --message "Look at this" --preview /path/to/local/image.jpeg
```
Preview images should be of about 250KB and within 512 to 512 pixels. There are
no hard limits, but clients might suppress posts if they exceed certain limits.

For a complete list of all post types and parameters, run:
```
$ spxp help post
```

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

#### Private encrypted posts
To create encrypted posts, you first need to create a new publishing group, for
example for your friends:
```
$ spxp group create friends
```
The group id (e.g. `friends`) is used as part of the protocol and has to comply
with certain limitations, like consisting purely of base64url safe characters.

Once you have created at least one group, you can encrypt any post with the
`--group` parameter:
```
$ spxp post text --message "Hello Friends" --group friends
```

If you want to allow recipients to respond to your post with a comment or a
reaction, you need to specify the group name in which these reactions need
to be published with the `--contribute` parameter:
```
$ spxp post text --message "Hello Friends" --group friends --contribute friends
```

#### Connection requests
You can use this tool to send connection requests, but it does not (yet) handle
the response coming back. It is still useful to test a client application, but
with limited use in real life.  
To request a connection, run the following command:
```
$ spxp connect request <profileUri> <group>
```
for example:
```
$ spxp connect request https://spxp.example.com/alice friends
```
This will send a connection request to the profileUri, giving this peer profile
read and write access to your profile.
