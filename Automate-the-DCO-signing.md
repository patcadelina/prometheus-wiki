All Prometheus repositories require DCO signing. By adding a _Signed-off-by_ line to your commit, you sign off [this Developer Certificate of Origin](https://developercertificate.org/). You don't have to add this line manually. You can simply add a `-s` flag to your `git commit` command. If you want to automate even this flag addition, follow the instructions below. __You must be fully aware that this automatically [certifies the origin of all your commits by default](https://developercertificate.org/).__

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