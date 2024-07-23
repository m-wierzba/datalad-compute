# DataLad compute extension

This is a naive datalad compute extension that serves as a playground for
the datalad remake-project.

It contains an annex remote that can compute content on demand. It uses template
files that specify the operations and per-data file parameters that are encoded
in annex URL-keys. It also contains the new datalad command `compute` that
can trigger the computation of content and store the parameters that are
used for content creation in the git-annex branch, where they can be used by
the annex remote to repeat the computation.


## Example usage

Install the extension, create a dataset, configure it to use `compute`-URLs


```bash
> datalad create compute-test-1
> cd compute-test-1
> git config annex.security.allowed-url-schemes compute
> git config annex.security.allowed-ip-addresses all
> git config annex.security.allow-unverified-downloads ACKTHPPT
```

Create the template directory and a template

```bash
> mkdir -p .datalad/compute/methods
> cat > .datalad/compute/methods/params_to_text <<EOF
inputs = ['points', 'label']
output = 'output_file'

use_shell = 'true'
executable = "echo"

arguments = [
    "generated via compute: ",
    "points: {points}, ",
    "label: {label}",
    ">",
    "'{output_file}'",
]
EOF
> datalad save -m "add compute method"
```

Create a "compute" annex special remote:
```bash
> git annex initremote compute encryption=none type=external externaltype=compute
```

Execute a computation and save the result:
```bash
> datalad compute params_to_text text-1.txt points=101 label=einhunderteins
```

The previous command will create the file `text-1.txt`:
```bash
> cat text-1.txt
generated via compute: points: 101, label: einhunderteins
```

Drop the content of `text-1.txt`, verify it is gone, recreate it via
`datalad get`, which "fetches" is from the compute remote:

```bash
> datalad drop text-1.txt
> cat text-1.txt
> datalad get text-1.txt
> cat text-1.txt
``` 

Generate a speculative computation, i.e. do not perform the computation but generate an
URL-KEY with associated URLs that describe the computation that should be performed. This
is done by giving the `-u` option (url-only) to `datalad compute`.

```bash
> datalad compute params_to_text -u text-2.txt points=303 label=dreihunderdrei
> cat text-2.txt      # this will fail, because the computation has not yet been performed
```

`ls -l text-2.txt` will show a link to a non-existent URL-KEY with a time-stamp identifier.
`git annex whereis text-2.txt` will show the associated URLs. No computation has been
performed yet, `datalad compute` just creates an URL-KEY and associates the method-,
parameters-, and dependencies-URLs with the URL-KEY.

Use `datalad get` to perform the computation for the first time and receive the result::
```bash
> datalad get text-2.txt
> cat text-2.txt
```


# Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) if you are interested in internals or
contributing to the project.
