---
title: "Package repositories"
linkTitle: "Package repositories"
date: 2019-12-14
description: >
  How to create Luet repositories
---

After a set of packages has been built, a repository must be created in order to make them accessible by Luet clients. A Repository can be served either local files or via http(s) (at the moment of writing). Luet, by default, supports multiple-repositories with priorities.

## Create a repository

```
Generate and renew repository metadata

Usage:
  luet create-repo [flags]

Flags:
      --descr string              Repository description (default "luet")
  -h, --help                      help for create-repo
      --meta-compression string   Compression alg: none, gzip (default "none")
      --meta-filename string      Repository metadata filename (default "repository.meta.yaml.tar")
      --name string               Repository name (default "luet")
      --output string             Destination folder (default current dir)
      --packages string           Packages folder (output from build) (default current dir)
      --reset-revision            Reset repository revision.
      --tree string               Source luet tree (default "/home/mudler/_git/luet/test")
      --tree-compression string   Compression alg: none, gzip (default "gzip")
      --tree-filename string      Repository tree filename (default "tree.tar")
      --type string               Repository type (disk) (default "disk")
      --urls strings              Repository URLs

Global Flags:
      --concurrency int   Concurrency (default 8)
      --config string     config file (default is $HOME/.luet.yaml)
      --fatal             Enables Warnings to exit
  -v, --verbose           verbose output

```

After issuing a `luet build`, the built packages are present in the output build directory. The `create-repo` step is needed to generate a portable tree, which is read by the clients, and a `repository.yaml` which contains the repository metadata.

Note that the output of `create-repo` is *additive* so it integrates with the current build content. The repository is composed by the `build` output command and the `create-repo` metadata. 

### Flags

- **--descr**: Repository description
- **--name**: Repository name
- **--output**: Metadata output folder (while a different path can be specified, it's prefered to output the metadata files directly to the package directory).This most of the time matches the packages path for convenience.
- **--packages**: Directory where built packages are stored. This most of the time is also the output path.
- **--reset-revision**: Reset the repository revision number
- **--tree-path**: Specify a custom name for the tree path. (Defaults to tree.tar)
- **--tree-compression**: Specify a compression algorithm for the tree. (Available: gzip, Defaults: none)
- **--tree**: Path of the tree which was used to generate the packages.
- **--type**: Repository type (http/local). It is just descriptive, the clients will be able to consume the repo in whatsoever way it is served.
- **--urls**: List of URIS where the repository is available

## Example

Build a package and generate the repository metadata:

```
$> mkdir package

$> cat <<EOF > package/build.yaml
image: busybox
steps:
- echo "foo" > /foo
EOF

$> cat <<EOF > package/definition.yaml
name: "foo"
version: "0.1"
category: "bar" # optional!
EOF

$> luet build --all --destination $PWD/out/ --tree $PWD/package

📦  Selecting  foo 0.1
📦  Compiling foo version 0.1 .... ☕
🐋  Downloading image luet/cache-foo-bar-0.1-builder
🐋  Downloading image luet/cache-foo-bar-0.1
📦   foo Generating 🐋  definition for builder image from busybox
🐋  Building image luet/cache-foo-bar-0.1-builder
🐋  Building image luet/cache-foo-bar-0.1-builder done
 Sending build context to Docker daemon  4.096kB
 ...

$> luet create-repo --name "test" --output $PWD/out --packages $PWD/out --tree $PWD/package
 For repository test creating revision 1 and last update 1580641614...

$> ls out
foo-bar-0.1-builder.image.tar  foo-bar-0.1.image.tar  foo-bar-0.1.metadata.yaml  foo-bar-0.1.package.tar  repository.yaml  tree.tar

```

