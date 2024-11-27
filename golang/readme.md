# How to Use a Private Go Module in Gitlab

## configure go env

```bash
go env -w GOPRIVATE="gitlab.example.com"
go env -w GOINSECURE="gitlab.example.com"
```

go get will only try ports 443 and 80.

<https://stackoverflow.com/questions/53780309/go-get-from-a-private-repo-using-a-non-standard-http-port>

<https://stackoverflow.com/questions/58305567/how-to-set-goprivate-environment-variable>

## go get by http

Choose either method1 or method2 for authentication.

- Method1: use `~/.netrc`  
    ```
    machine gitlab.example.com
    login caesar
    password 1988771017
    ```

    ```
    machine <url>
    login <username>
    password <token or password>
    ```

- Method2: use `~/.gitconfig`  
    ```bash
    git config --global url.http://caesar:1988771017@gitlab.example.com/.insteadOf http://gitlab.example.com/
    ```

    ```bash
    git config --global url."https://${user}:${personal_access_token}@gitlab.example.com".insteadOf "https://gitlab.example.com"
    ```

<https://docs.gitlab.com/ee/user/project/working_with_projects.html#authenticate-go-requests-to-private-projects>

<https://gist.github.com/serkodev/1763e0d8733c9810cc6ab9d2c335e417#authentication>

## go get by ssh

1. set `~/.gitconfig`
    ```bash
    git config --global url.ssh://git@gitlab.example.com/.insteadOf http://gitlab.example.com/
    ```

2. add ssh key to gitlab
    ```bash
    $ ssh-keygen -f ~/.ssh/key_gitlab -C "caesar.t@hotmail.com"

    # Check if ssh-agent is running properly.
    $ eval $(ssh-agent -s)

    $ ssh-add ~/.ssh/key_gitlab
    ```

3. set `~/.ssh/config`
    ```
    Host gitlab.example.com
      HostName gitlab.example.com
      User caesar
      IdentityFile ~/.ssh/key_gitlab
      Port 2224
    ```

    ```
    Host <alias>
      HostName <your-gitlab-host>
      User <your-username>
      IdentityFile <key path>
      Port <port-number>
    ```

## deploy key for automation bot

```
ssh-keygen -f ~/.ssh/golang_ssh -C "used to golang get private package"
```

for `common` repository:  
using public ssh key  

for `go service` repository using `common`:  
using private ssh key  

<https://docs.gitlab.com/ee/user/project/deploy_keys/index.html#create-a-project-deploy-key>

<https://gitlab.com/x246libra/IsCoolLab2024/-/blob/main/pipelines/vars.yml?ref_type=heads#L136-149>

## fix subgroup issue on gitlab

### Error output:
```bash
# go get by ssh
$ go get gitlab.example.com/project1/backend/util

go: module gitlab.example.com/project1/backend/util: git ls-remote -q origin in /home/caesar/.gvm/pkgsets/go1.20/global/pkg/mod/cache/vcs/ba9abb22a30b8a743877742bd2f0d2d0421f68bcf0b039654ad775d18d368ace: exit status 128:
        remote: 
        remote: ========================================================================
        remote: 
        remote: The project you were looking for could not be found or you don't have permission to view it.
        remote: 
        remote: ========================================================================
        remote: 
        fatal: Could not read from remote repository.

        Please make sure you have the correct access rights
        and the repository exists.
```

```bash
# go get by http
$ go get gitlab.example.com/project1/backend/util

go: module gitlab.example.com/project1/backend/util: git ls-remote -q origin in /home/caesar/.gvm/pkgsets/go1.20/global/pkg/mod/cache/vcs/ba9abb22a30b8a743877742bd2f0d2d0421f68bcf0b039654ad775d18d368ace: exit status 128:
        remote: The project you were looking for could not be found or you don't have permission to view it.
        fatal: repository 'http://gitlab.example.com/project1/backend.git/' not found
```

### The reason for the issue

```
# If a project is found and the user has access, we return the full project path
# If not, we return the first two components as if it were a simple `namespace/project` path,
# so that we don't reveal the existence of a nested project the user doesn't have access to.
# This means that for an unauthenticated request to `group/subgroup/project/subpackage`
# for a private `group/subgroup/project` with subpackage path `subpackage`, GitLab will respond
# as if the user is looking for project `group/subgroup`, with subpackage path `project/subpackage`.
# Since `go get` doesn't authenticate by default, this means that
# `go get gitlab.com/group/subgroup/project/subpackage` will not work for private projects.
# `go get gitlab.com/group/subgroup/project.git/subpackage` will work, since Go is smart enough
# to figure that out. `import 'gitlab.com/...'` behaves the same as `go get`.
```

<https://gitlab.com/gitlab-org/gitlab-foss/-/issues/37832#note_52391221>

### Solution

- method 1:  
    Using gitlab website, change the project's visibility to "public"

- method 2:  
    replace syntax in `go.mod`

    ```
    require (
      gitlab.com/org/subgroup/repo v0.0.1
    )
    replace gitlab.com/org/subgroup/repo => gitlab.com/org/subgroup/repo.git v0.0.1
    ```

- method 3:  
    rename module name in `go.mod`

    ```
    before:
    module gitlab.example.com/project1/backend/util

    after:
    module gitlab.example.com/project1/backend/util.git
    ```

open issue:  
<https://gitlab.com/gitlab-org/gitlab/-/issues/36354>

close issue:  
<https://github.com/golang/go/issues/29888>

<https://gitlab.com/gitlab-org/gitlab-foss/-/issues/30785>

<https://gitlab.com/gitlab-org/gitlab-foss/-/issues/45055>

<https://docs.gitlab.com/16.4/ee/user/public_access.html>
