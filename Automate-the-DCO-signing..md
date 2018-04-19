- Set a hook to sign all commits.

To Set for a single repo create the hook file in: `/project/root/.git/hooks/`<br/>
Or set a global  hook directory which will be used for all repos.
`git config --global core.hooksPath /path/to/gitHooks`

- Than create the hook file.<br/>
`nano /path/to/gitHooks/prepare-commit-msg`<br/>
or<br/>
`nano /project/root/.git/hooks/prepare-commit-msg`



```
#!/bin/sh

NAME=$(git config user.name)
EMAIL=$(git config user.email)

if [ -z "$NAME" ]; then
    echo "empty git config user.name"
    exit 1
fi

if [ -z "$EMAIL" ]; then
    echo "empty git config user.email"
    exit 1
fi

git interpret-trailers --if-exists doNothing --trailer \
    "Signed-off-by: $NAME <$EMAIL>" \
    --in-place "$1"
```