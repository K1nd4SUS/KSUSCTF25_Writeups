As the description says we have some control over the execution in the container:
- an environment variable
- a python script.
Our control is limited by the many tests on the input and the `-I` (isolated mode) flag.
The "stick guy floating around" is a reference to XKCD comic number 353.
(Note that the server port ended with 353, just for verification)
One of Python easter eggs is the module `antigravity` (a convenient solution to a gravity well).
That module opens a webpage to the comic.
The imported `webbrowser` module uses the `BROWSER` environment variable.
That will be our attack point, payload:
`sh -c "cat flag.txt; echo %s"`
while the script can be the innocent looking
`import antigravity`
