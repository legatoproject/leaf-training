- [Build a simple package](#build-a-simple-package)
- [Details of the manifest.json](#details-of-the-manifestjson)
  - [The 'info' node](#the-info-node)
  - [Steps](#steps)
  - [Settings](#settings)
  - [Env variables](#env-variables)
  - [Binaries](#binaries)
  - [Plugins](#plugins)
  - [Help topics](#help-topics)
- [Full example: Building an SDK](#full-example-building-an-sdk)
  - [Packaging a tool: mytool](#packaging-a-tool-mytool)
  - [Packaging a lib: mylib](#packaging-a-lib-mylib)
  - [Packaging a SDK: mysdk](#packaging-a-sdk-mysdk)
  - [Build the remote and test the SDK](#build-the-remote-and-test-the-sdk)
- [Special dependency: *requires*](#special-dependency-requires)
  - [Example: license package](#example-license-package)
  - [Other use cases](#other-use-cases)
- [Conditional Dependencies](#conditional-dependencies)
  - [Update the SDK](#update-the-sdk)
  - [Declare a setting](#declare-a-setting)
- [Package auto upgrade for tools](#package-auto-upgrade-for-tools)
- [Extra features](#extra-features)
  - [Leaf build manifest: *layers*](#leaf-build-manifest-layers)
  - [The generated *leaf.info* file](#the-generated-leafinfo-file)
  - [Server side tags: *leaf.info.tags*](#server-side-tags-leafinfotags)
  - [Tweak the tar generation](#tweak-the-tar-generation)


# Build a simple package

Generate a manifest
```sh
leaf build manifest --help
leaf build manifest --name myapp --version 1.0
```

Here is the generated `manifest.json`
```json
{
  "info": {
    "name": "myapp",
    "version": "1.0"
  }
}
```

Build the package
```sh
leaf build pack  --help
leaf build pack -i manifest.json -o myapp_1.0.leaf
```

Best practice: *one folder per package and build outside the folder*
```sh
mkdir myapp_1.0 && mv manifest.json myapp_1.0
leaf build pack -i myapp_1.0 -o myapp_1.0.leaf
```

Install myapp
```sh
leaf package install ./myapp_1.0.leaf
leaf package uninstall myapp_1.0
```

Build a remote
```sh
leaf build index --help
leaf build index --prettyprint -o index.json *.leaf
```

Test it
```sh
leaf remote add myrepo --insecure $PWD/index.json
leaf search -a
```


# Details of the manifest.json

## The 'info' node

```json
{
  "info": {
    "name": "mysdk",
    "version": "1.0",
    "description": "Some description for our SDK",
    "documentation": "https://myapp.leaf.io/",
    "master": true,
    "date": "2019-08-09 16:04:00",
    "tags": ["dev", "beta"],
    "depends": ["myapp_1.0", "mylib_1.0", "mytool_1.0"],
    "requires": ["mylicense_1.0"],
    "leafMinVersion": "2.1"
  }
}
```

> You retrieve the info node in the *index.json* 

## Steps

Basic install step

```json
{
  "info": {
    "name": "myapp",
    "version": "1.0"
  },
  "install": [
    {
      "command": [
          "echo",
          "Hello World"
      ]
    }
  ]
}
```

More complex install steps
```json
{
    "info": {
        "name": "myapp",
        "version": "1.0"
    },
    "install": [
        {
            "command": [
                "echo",
                "Hello $FOO"
            ],
            "env": {
                "FOO": "World :)"
            },
            "verbose": true
        },
        {
            "label": "Some long operation ...",
            "command": [
                "sleep",
                "3"
            ]
        },
        {
            "command": [
                "false"
            ],
            "ignoreFail": true
        },
        {
            "label": "Generate random data...",
            "command": [
                "dd",
                "if=/dev/urandom",
                "of=@{DIR}/data.random",
                "bs=1M",
                "count=$MY_COUNT"
            ],
            "env": {
                "MY_COUNT": "100"
            },
            "verbose": true
        }
    ]
}
```

Other steps 

```json
{
    "info": {
        "name": "myapp",
        "version": "1.0"
    },
    "install": [],
    "uninstall": [],
    "sync": []
}
```

## Settings

Settings are env variable stored in user/workspace/profile scope.
Example below.

## Env variables
```json
{
    "info": {
        "name": "myapp",
        "version": "1.0"
    },
    "env": {
        "FOO": "BAR",
        "PATH": "$PATH:@{DIR}/bin",
        "ADDRESS": "${ADDRESS:-10.0.0.9}"
    }
}
```

## Binaries
```json
{
    "info": {
        "name": "myapp",
        "version": "1.0"
    },
    "bin": {
        "mybin": {
         "path": "@{DIR}/mybin.sh"
        },
        "touch": {
            "path": "touch",
            "description": "touch a file"
        }
    }
}
```

## Plugins
```json
{
    "info": {
        "name": "myapp",
        "version": "1.0"
    },
    "plugins": {
        "myplugin": {
            "source": "myplugin.py",
            "description": "some usefull extension for leaf"
        }
    }
}
```

## Help topics
```json
{
    "info": {
        "name": "myapp",
        "version": "1.0"
    },
    "help": {
        "myapp": {
            "html": "https://github.com/legatoproject/leaf"
        },
        "mylib": {
            "html": "https://github.com/legatoproject/leaf",
            "txt": "somefile.txt"
        }
    }
}
```
# Full example: Building an SDK

## Packaging a tool: mytool

```sh
mkdir mytool_1.0 && cd mytool_1.0
leaf build manifest --name mytool --version 1.0
mkdir bin && cp $(which fortune) bin/mytool
```

Update the `$PATH`
```json
{
    "info": {
        "name": "mytool",
        "version": "1.0"
    },
    "env": {
        "PATH": "$PATH:@{DIR}/bin/"
    }
}
```

Build the package
```sh
leaf build pack -i mytool_1.0 -o mytool_1.0.leaf
```

Test it
``` sh
leaf package install ./mytool_1.0.leaf
leaf env package mytool_1.0
zsh
eval "$(leaf env package mytool_1.0)"
mytool
exit
```

## Packaging a lib: mylib

```sh
mkdir mylib_1.0 && cd mylib_1.0
leaf build manifest --name mylib --version 1.0
mkdir include && cp -av /usr/include/openssl/ include/
```

Update the `$C_INCLUDE_PATH`
```json
{
    "info": {
        "name": "mylib",
        "version": "1.0"
    },
    "env": {
        "C_INCLUDE_PATH": "$C_INCLUDE_PATH:@{DIR}/include/"
    }
}
```

Build the package
```sh
leaf build pack -i mylib_1.0 -o mylib_1.0.leaf
```

## Packaging a SDK: mysdk

```sh
mkdir mysdk_1.0 && cd mysdk_1.0
leaf build manifest --name mysdk --version 1.0
```

Edit the `manifest.json`
```json
{
    "info": {
        "name": "mysdk",
        "version": "1.0",
        "master": true,
        "depends": [
            "myapp_1.0",
            "mylib_1.0",
            "mytool_latest"
        ]
    }
}
```

Build the package
```sh
leaf build pack -i mysdk_1.0 -o mysdk_1.0.leaf
```

## Build the remote and test the SDK

```sh
leaf build index --prettyprint -o index.json *.leaf
leaf remote fetch
leaf search -a
```

Time to start a `Makefile`
```
.PHONY: clean

MY_PACKAGES:= \
	myapp_1.0.leaf \
	mylib_1.0.leaf \
	mytool_1.0.leaf \
	mysdk_1.0.leaf

all: index.json
	leaf remote fetch

index.json: $(MY_PACKAGES)
	leaf build index --prettyprint -o index.json $^

%.leaf: %/manifest.json
	leaf build pack -o $@ -i $<

clean:
	rm -f *.leaf *.leaf.info index.json
```

Install the SDK
```sh
leaf setup -p mysdk
leaf shell
leaf env 
env | grep INCLUDE
mytool
exit
```

# Special dependency: *requires* 

## Example: license package

```sh
mkdir mylicense_1.0 && cd mylicense_1.0
leaf build manifest --name mylicense --version 1.0
```

Let's write a license accepter: `accept.sh`
```sh
#/bin/sh
set -e

echo "Here is my license:
bla bla bla ...
Do you accept it? (y/n)"

test "$LEAF_NON_INTERACTIVE" = "1" && exit 2
read RESP
test "$RESP" = "y"
```

```sh
chmod +x accept.sh
```

Create a `sync` step in the `manifest.json`
```json
{
    "info": {
        "name": "mylicense",
        "version": "1.0"
    },
    "sync": [
        {
            "verbose": true,
            "command": [
                "./accept.sh"
            ]
        }
    ]
}
```

Update the SDK manifest
```json
{
    "info": {
        "name": "mysdk",
        "version": "1.0",
        "master": true,
        "depends": [
            "myapp_1.0",
            "mylib_1.0",
            "mytool_latest"
        ],
        "requires": [
            "mylicense_latest"
        ]
    }
}
```

Update the Makefile to build the new package
```sh
...
MY_PACKAGES:= \
	myapp_1.0.leaf \
	mylib_1.0.leaf \
	mytool_1.0.leaf \
	mysdk_1.0.leaf \
	mylicense_1.0.leaf
...
```

And try it
> Clean leaf installed package and current workspace before
```sh
leaf package install mysdk_1.0
```

## Other use cases

The *prereq* packages are installed before any further installation, this can be useful to check *apt* dependencies for example.



# Conditional Dependencies

*Conditional Dependencies* are dependencies that test environment variables to resolve the dependency tree:
- `FOO`: *FOO* is set
- `!FOO`: *FOO* is not set
- `FOO=BAR`: *FOO* equals 'BAR'
- `FOO!=BAR`: *FOO* is not 'BAR'
- `FOO~bar`: *FOO* contains 'bar' (ignore case)
- `FOO!~bar`: *FOO* does not contain 'bar' (ignore case)

## Update the SDK

```json
{
    "info": {
        "name": "mysdk",
        "version": "1.0",
        "master": true,
        "depends": [
            "myapp_1.0",
            "mylib_1.0(DEVEL_MODE=1)",
            "mytool_latest"
        ],
        "requires": [
            "mylicense_latest"
        ]
    }
}
```

> Clean leaf installed package and current workspace before

Let's use the SDK now
```sh
leaf setup MY_PROFILE -p mysdk
leaf status -v
```

```sh
leaf setup MY_PROFILE_DEV -p mysdk
leaf status -v
leaf env profile --set DEVEL_MODE=1
leaf status -v
leaf profile sync
```

## Declare a setting

Update the SDK
```json
{
    "info": {
        "name": "mysdk",
        "version": "1.0",
        "master": true,
        "depends": [
            "myapp_1.0",
            "mylib_1.0(DEVEL_MODE=1)",
            "mytool_latest"
        ],
        "requires": [
            "mylicense_latest"
        ]
    },
    "settings": {
        "devel.mode": {
            "key": "DEVEL_MODE",
            "description": "Set the devel mode for mysdk",
            "regex": "[01]",
            "scopes": [
                "user",
                "workspace",
                "profile"
            ]
        }
    }
}
```
> Clean leaf installed package and current workspace

```sh
leaf setup MY_PROFILE -p mysdk
leaf status -v
leaf setup MY_PROFILE_DEV -p mysdk
leaf config list devel -v
leaf config set mysdk.devel.mode 1 --profile
leaf status -v
leaf profile sync
```

# Package auto upgrade for tools

Update *mytool* `manifest.json`

```json
{
    "info": {
        "name": "mytool",
        "version": "1.0",
        "upgrade": true
    },
    "env": {
        "PATH": "$PATH:@{DIR}/bin/"
    }
}
```

Create a new version of *mytool*
```sh
cp -a mytool_1.0 mytool_2.0
```

And update the `manifest.json`
```json
{
    "info": {
        "name": "mytool",
        "version": "2.0",
        "upgrade": true
    },
    "env": {
        "PATH": "$PATH:@{DIR}/bin/"
    },
    "bin": {
        "mycommand": {
            "path": "@{DIR}/bin/mytool",
            "description": "Tells you something"
        }
    }
}
```

Update the Makefile
```sh
...
MY_PACKAGES:= \
	myapp_1.0.leaf \
	mylib_1.0.leaf \
	mytool_1.0.leaf \
	mytool_2.0.leaf \
	mysdk_1.0.leaf \
	mylicense_1.0.leaf
...
```

> Clean leaf installed package and current workspace

Test it
```sh
leaf package install mytool_1.0
leaf package list -a
leaf run 

leaf package upgrade
leaf package list -a 
leaf run
leaf run mycommand
```

There is an arg `--clean`
```sh
leaf package uninstall mytool_2.0
leaf package list -a
leaf run 

leaf package upgrade --clean
leaf package list -a 
leaf run
leaf run mycommand
```

# Extra features

## Leaf build manifest: *layers*

Create a new version of `myapp`

```sh
mkdir myapp_2.0 && cd myapp_2.0
```
Now create some layers:

- file `myapp_2.0/layer-info.json`
```json
{
    "info": {
        "description": "Some description here",
        "documentation": "https://github.com/legatoproject/leaf"
    }
}
```
- file `myapp_2.0/layer-install.json`
```json
{
    "install": [
        {
            "command": [
                "echo",
                "Hello World"
            ]
        }
    ]
}
```

Build a manifest:
```sh
leaf build manifest --append layer-info.json --append layer-install.json --name myapp --version 2.0
cat manifest.json
# CLI arguments orverride fields
leaf build manifest --append layer-info.json --append layer-install.json --name myapp --version 2.0 --description "My cool app"
cat manifest.json
```

Update the `Makefile` to build the `myapp_2.0/manifest.json` automatically:
```txt
.PHONY: clean

MY_PACKAGES:= \
	myapp_1.0.leaf \
	myapp_2.0.leaf \
	mylib_1.0.leaf \
	mytool_1.0.leaf \
	mytool_2.0.leaf \
	mysdk_1.0.leaf \
	mylicense_1.0.leaf

all: index.json
	leaf remote fetch

index.json: $(MY_PACKAGES)
	leaf build index --prettyprint -o index.json $^

myapp_2.0.leaf: myapp_2.0/layer-info.json myapp_2.0/layer-install.json
	leaf build manifest \
		--append myapp_2.0/layer-info.json \
		--append myapp_2.0/layer-install.json \
		--name myapp --version 2.0 --description "My cool app" \
		-o myapp_2.0/
	leaf build pack -o $@ -i myapp_2.0

%.leaf: %/manifest.json
	leaf build pack -o $@ -i $<

clean:
	rm -f *.leaf *.leaf.info index.json
```

## The generated *leaf.info* file

This file contains metadata to speed up indexing operations. This file is optionnal, while creating an index, if the `XXX.leaf.info` does not exist, *leaf* will recompute the *sha384* which is time consuming.
> See `leaf build pack --no-info` not to generate the *info* files
```json
{
    "info": {
        "name": "myapp",
        "version": "1.0"
    },
    "hash": "sha384:7cceefacbe26022e3ae925ebccdde03aabc31fbd7e4befaa34285c26264cef090eb9967ae42a1f5cf7fde13532bf37e7",
    "size": 10240
}
```

## Server side tags: *leaf.info.tags*

Create a *server-side* tag file `myapp_2.0.leaf.tags` which contains:
```txt
internal
dev
```

Update the `myapp_2.0/layer-info.json`
```json
{
    "info": {
        "description": "Some description here",
        "documentation": "https://github.com/legatoproject/leaf",
        "tags": ["signed", "beta"]
    }
}
```

Rebuild the repository using `make`

Tags `internal` and `dev` are only visible with command `search` because they are only present on server side (in the `index.json`) not on client side (in the package `manifest.json` once package installed):
```sh
$ leaf search --all --verbose --tag beta
┌───────────────────────────────────────────────────────────────────┐
│                     1 package - Filter: +beta                     │
├────────────┬──────────────────────────────────────────────────────┤
│ Identifier │                      Properties                      │
╞════════════╪══════════════════════════════════════════════════════╡
│ myapp_2.0  │   Description: My cool app                           │
│            │ Documentation: https://github.com/legatoproject/leaf │
│            │          Tags: latest,signed,beta,internal,dev       │
│            │        Source: myrepo                                │
│            │          Size: 10 kB                                 │
└────────────┴──────────────────────────────────────────────────────┘

$ leaf package install myapp_2.0
[...]

$ leaf package list --all --verbose --tag beta
┌───────────────────────────────────────────────────────────────────┐
│                     1 package - Filter: +beta                     │
├────────────┬──────────────────────────────────────────────────────┤
│ Identifier │                      Properties                      │
╞════════════╪══════════════════════════════════════════════════════╡
│ myapp_2.0  │   Description: My cool app                           │
│            │ Documentation: https://github.com/legatoproject/leaf │
│            │          Tags: signed,beta                           │
│            │        Folder: /home/seb/.leaf/myapp_2.0             │
└────────────┴──────────────────────────────────────────────────────┘
```

## Tweak the tar generation

```sh
leaf build pack --help
[...]
examples: 
  Build an GZIP compressed archive
    $ leaf build pack -i path/to/packageFolder/ -o package.leaf -- -z .
  Build an XZ compressed archive with some files excluded (from /tmp/exclude.list), tar will be verbose
    $ leaf build pack -i path/to/packageFolder/ -o package.leaf -- -v -J -X /tmp/exclude.list .
  Build an XZ compressed archive containing only the manifest.json file
    $ leaf build pack -i path/to/packageFolder/ -o package.leaf -- manifest.json
```

```sh
dd if=/dev/zero of=myapp_2.0/data.bin bs=1M count=100
time leaf build pack -i myapp_2.0 -o myapp_2.0.leaf
time leaf build pack -i myapp_2.0 -o myapp_2.0.leaf.gz -- -z .
time leaf build pack -i myapp_2.0 -o myapp_2.0.leaf.xz -- -J .
ls -hl --color myapp_2.0.leaf*
```
> The package extension is not mandatory, *leaf* will automatically guess the compression without relying on the extension.
