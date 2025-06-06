# Downloading Dropbox Files in Colab

How do you get a link that curl can use if you're viewing a file too large to preview in a shared folder?

One way is to download it from your browser, go to Ctrl+J, and then get the link from there.
This worked at one point but gave me a 403 error another time (though I think I know what went wrong but still need to check).

Another way is to use the Dropbox Python SDK.

`pip install dropbox`

then (following https://stackoverflow.com/questions/71258484/download-a-single-file-from-a-shared-dropboxs-folder-without-having-the-share-l):

```py
import dropbox
dbx = dropbox.Dropbox(...)
...
res = dbx.sharing_get_shared_link_file(url=shared_folder_URL, path="/myfile.zip")
curl_url = res[0].url
```

Then that curl_url can be passed to `curl -L --output filename.zip <curl_url>` and the file download starts.

I still need to see if the above _requires_ setting up an access token in Dropbox if the file is public.
The fact that I tried first creating an access token for an app with "app folder" access, and it didn't work,
but creating one for an app with "full Dropbox" access did work, suggests it _is_ required.
Note: it _may_ be possible with "app folder" access if I added more permissions, since I added a few more
to my "full Dropbox" app than my "app folder" app and didn't keep experimenting. But the StackOverflow user said there's a bug that caused the "app folder" access type to not work for this; they updated their post later to say the bug's been fixed, but maybe it came back or the fix isn't in the version I can get with pip in Colab or something.

I did it to narrow down troubleshooting issues to "be safe", but it's annoying.

If you do have to make a token, then you do so by going to https://www.dropbox.com/developers/ and creating an "app".
Then in the "Settings" for said app there's a button for generating access tokens.

In place of using the Python library, one could also use their access token in a curl POST command as described elsewhere in the Dropbox API docs.

This example on StackOverflow might also be useful: https://stackoverflow.com/questions/74055519/how-can-write-curl-command-to-upload-file-into-dropbox-with-refresh-code

One advantage of the API is that for non-public shares/files, your own files, etc. it will wok whereas the "Ctrl+J" browser method will not. However, a different approach from the Dropbox API (and implied token creation) could be DSFLJKSDFLJK
