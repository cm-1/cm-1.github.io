# .tar

Links that cover the structure of a tar file:

- https://depot.dev/blog/what-is-a-tar-file
- https://jackrabbit.apache.org/oak/docs/nodestore/segment/tar.html
- https://www.ibm.com/docs/en/zos/3.1.0?topic=formats-tar-format-tar-archives

The [gnu.org page on it](https://www.gnu.org/software/tar/manual/html_node/Standard.html) contains a lot more depth.
Some of the `#define` statements seem helpful, e.g., for the typeflag field. However, it also talks a fair bit about sparse files and sparse headers and I'm _guessing_ that's usually not something one has to worry about?


## See Also

I have separate notes on how to extract from split-up .tar files, how to extract from stdout, etc. but figured those notes made more sense in my Linux notes rather than file format notes.
