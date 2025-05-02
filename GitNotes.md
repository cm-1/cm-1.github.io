# Git Stash
If "git stash" is failing with an error message mentioning a file being "not uptodate", a possible workaround is staging/adding all files and *then* stashing.

# Git Revert File
`git checkout HEAD -- myfile.txt`
(as described https://stackoverflow.com/questions/7147270/hard-reset-of-a-single-file)

# Git Unstage File
`git reset -- myfile.txt`
Do **NOT** use `git rm`, as that actually **stages the DELETION of the file!**
(https://stackoverflow.com/questions/6919121/why-there-are-two-ways-to-unstage-a-file-in-git)

# Locally ignore files without editing .gitignore
Let's say you have some notes or something that you don't want to add to the git repo but that you don't want to "pollute" the .gitignore by mentioning, since nobody else
will probably need to worry about ignoring files with that name/path/pattern.

What you can then do is to edit `<your project dir>/.git/info/exclude` file ("exclude" is a text file without an extension, not a directory name) with the names of the files that you want to exclude!

This comes from https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files.
The link also mentions a global computer-wide option, but that sounds more error-prone and more likely to be unnecessary.

# "How to get just one file from another branch"

https://stackoverflow.com/questions/2364147/how-to-get-just-one-file-from-another-branch

```
git switch dest_branch
git restore --source source_branch -- thing.txt
```
