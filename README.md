# <https://www.robertdeveen.com>

## Install
See: https://jekyllrb.com/docs/installation/ubuntu/

```bash
sudo apt-get install ruby-full build-essential zlib1g-dev ruby-rubygem ruby-dev ubuntu-dev-tools
gem install jekyll bundler
```

### WSL Fix
See: https://stackoverflow.com/questions/57243299/jekyll-operation-not-permitted-apply2files

```bash
sudo -
cat <<'EOF' > /etc/wsl.conf
[automount]
enabled = true
root = /mnt
options = "metadata,uid=1000,gid=1000,umask=022,case=off"
mountFsTab = false
EOF
```



## Run locally on WSL

```bash
bundle install
bundle exec jekyll serve --baseurl="" --livereload --force_polling --no-watch

or 

sudo apt install jekyll
jekyll serve --baseurl="" --livereload --force_polling --no-watch
```

Debugging:
In Dev Container run:
```bash
rdbg --command --open -- bundle exec jekyll serve --baseurl="" --livereload --force_polling --no-watch
```
And start the debugger in VSCode.