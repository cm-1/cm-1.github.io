# TAR

Links that cover the structure of a tar file:

- https://depot.dev/blog/what-is-a-tar-file
- https://jackrabbit.apache.org/oak/docs/nodestore/segment/tar.html
- https://www.ibm.com/docs/en/zos/3.1.0?topic=formats-tar-format-tar-archives

The [gnu.org page on it](https://www.gnu.org/software/tar/manual/html_node/Standard.html) contains a lot more depth.
Some of the `#define` statements seem helpful, e.g., for the typeflag field. However, it also talks a fair bit about sparse files and sparse headers and I'm _guessing_ that's usually not something one has to worry about?

Then [there's some Golang code on GitHub](https://github.com/AQUAOSOTech/tarsplitter/blob/master/tarsplitter.go) that splits
up a .tar file into separate ones in a way where no file is split between two splitted files. This is _probably_ not useful for my current use case, since my issue is that I am _starting_ from part files and cannot really make a giant cat out of them to resplit, but it _might_ be useful in the future, if for other reasons.

