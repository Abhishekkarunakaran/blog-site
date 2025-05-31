+++
date = '2025-05-30T23:40:27+05:30'
title = 'How I Released a Weekend Go Project Within Minutes.'
+++

> disclaimer : 
> This is not a tutorial. It's just a `story` how I released a project within minutes.

## Intro<span style="color:#87af5f">.</span>

Let me set the context first. While working on my day job, I ran into a simple need: converting a `UUID` to `base64`. I searched for online tools, but none of them did exactly what I wanted.

> Note:
> Yeah, I know now—you can just use a simple terminal command:
> ![](/blog-001/terminal.png)

But at the time, I needed a quick solution. So, I wrote a small Go script to convert a `UUID` to `base64`:

```go
package main

import (
	"encoding/base64"
	"fmt"
	"os"
	"github.com/gofrs/uuid"
)

func main() {
	uuidString := os.Args[1]

	uuidVal,err := uuid.FromString(uuidString)
	if err != nil {
		fmt.Println(err.Error())
		return
	}

	b64 := base64.StdEncoding.EncodeToString(uuidVal.Bytes())
	fmt.Printf("base64: %s",b64)
}
```
output:
![output](/blog-001/out.png)

Then it hit me,<i> "What not turn this into a TUI app?"</i>. At the time, I was exploring <a href="https://github.com/charmbracelet/bubbletea" target=_blank>bubbletea</a> for a side project.

The project was done in 24 hours—it didn’t take much effort. Here's the project: <a href="https://github.com/Abhishekkarunakaran/ub2" target=_blank>ub2</a>.

Since I needed it on my work laptop and it's a terminal app, using `go install` was the better choice.

And that brought me to the real question: "How do I release this properly?"

## Goreleaser<span style="color:#87af5f">.</span>

I did a quick google search, and <a href=https://goreleaser.com target=_blank>goreleaser</a> popped up.

It's pretty interesting: A cli app that helps you to release your `Go`,`Zig`,`Rust`,`Python` and`Typescript` projects.

### How to start ?

After installing the `goreleaser`, cd into your project directly and use the following command :
![init](/blog-001/init.png)

A `.goreleaser.yaml` file is created in the root directory.

```yaml
# This is an example .goreleaser.yml file with some sensible defaults.
# Make sure to check the documentation at https://goreleaser.com

# The lines below are called `modelines`. See `:help modeline`
# Feel free to remove those if you don't want/need to use them.
# yaml-language-server: $schema=https://goreleaser.com/static/schema.json
# vim: set ts=2 sw=2 tw=0 fo=cnqoj

version: 2

before:
  hooks:
    # You may remove this if you don't use go modules.
    - go mod tidy
    # you may remove this if you don't need go generate
    - go generate ./...

builds:
  - env:
      - CGO_ENABLED=0
    goos:
      - linux
      - windows
      - darwin

archives:
  - formats: [tar.gz]
    # this name template makes the OS and Arch compatible with the results of `uname`.
    name_template: >-
      {{ .ProjectName }}_
      {{- title .Os }}_
      {{- if eq .Arch "amd64" }}x86_64
      {{- else if eq .Arch "386" }}i386
      {{- else }}{{ .Arch }}{{ end }}
      {{- if .Arm }}v{{ .Arm }}{{ end }}
    # use zip for windows archives
    format_overrides:
      - goos: windows
        formats: [zip]

changelog:
  sort: asc
  filters:
    exclude:
      - "^docs:"
      - "^test:"

release:
  footer: >-

    ---

    Released by [GoReleaser](https://github.com/goreleaser/goreleaser).
```

This file is enough for the release if the goal is simply install the project using `go install`.

Now we need `GITHUB_TOKEN` and `tag`.

#### **Github Token:**

For generating token from github, you can refer <a href="https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens" target="_blank">this guide</a>.

**Step 1: Store the Token** <br>
Create a `.env.sh` file to store the token using this format:
```bash
export GITHUB_TOKEN="<github.token>"
```
> Note : Treat the token like a password -- Do not share it! <br>
If`.env.sh` is in the root directory, make sure to add it to `.gitignore` to avoid accidentally committing it.

**Step 2: Using the Token**<br>
You can use this github token two ways:
1. **Source the `.env.sh` file.**
```sh
source .env.sh
```
This makes the token available as an environment variable in the current terminal session. Any program running in this session can access it via `$GITHUB_TOKEN`.

2. **Add `env_file` in .goreleaser.yaml` file.** <br>

Add this line to your `.goreleaser.yaml`
```yaml
env_files:
  github_token: ./.env.sh
```

#### **Tags:**

Adding tag is pretty straight forward. After the first commit pushed to the remote ( here it is `github`).

**Step 1:Create Tag**
```sh
git tag -a v0.1.0 -m "message"
```
> Definition:<br>
For versioning, we normally uses this format :- `vx.y.z` <br>
x : Major version ( For breaking changes ) <br>
y : Minor version ( For backwards compatible changes ) <br>
z : Patch version ( For bug fixes )

**Step 2:Push Tag to Remote**
```sh
git push origin v0.1.0
```

Once everything is set up, it all comes down to a single command.

```sh
goreleaser release --clean
```

That’s it!
Your software is now out in the world!

> Note: `--clean` is used to delete the `dist` directory, where the builds exists. You can skip the flag for the first release. 

You can find the released zip files in the `Releases` section in the your github repo screen.
![releases](/blog-001/releases.png)

**Installing using go :**

Now you can install the app using go by simpling using the command:
```sh
go install github.com/Abhishekkarunakaran/ub2@latest
```
`goreleaser` even helped me set up a Homebrew tap so the tool can now be installed via brew—but that's a story for another post.

## Conclusion<span style="color:#87af5f">.</span>

If you're building CLI tools in Go, `goreleaser` is a no-brainer. It turns your local script into a downloadable, installable project with almost zero overhead.




