# Blog using Jekyll

## Install

Instal RVM (use the gpg command from https://rvm.io/rvm/install first):
```
curl -sSL https://get.rvm.io | bash -s stable --ruby
source /Users/goomad01/.rvm/scripts/rvm
export GEM_HOME="$HOME/.gem"  # add to .zshrc or .bashrc
gem install jekyll bundler
bundle install
```

Note: RVM needs to be uninstalled before exporting `GEM_HOME`, if you need to reinstall RVM, run `unset GEM_HOME` first.

If you encounter the following error

```
ERROR:  Error installing jekyll:
	ERROR: Failed to build gem native extension.
    current directory: /Users/[USER]/.gem/ruby/3.0.0/gems/eventmachine-1.2.7/ext

...

compiling binder.cpp
In file included from binder.cpp:20:
./project.h:119:10: fatal error: 'openssl/ssl.h' file not found
#include <openssl/ssl.h>
         ^~~~~~~~~~~~~~~
1 error generated.
make: *** [binder.o] Error 1

make failed, exit code 2

```

You need the following environment variables

```
export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"
export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"
export PKG_CONFIG_PATH="/usr/local/opt/openssl@1.1/lib/pkgconfig"
```

## Usage

```
bundle exec jekyll server
```

If you encounter the following error

```
`require': cannot load such file -- webrick (LoadError)
```

Then you need to run

```
bundle add webrick
```

## Alternatively, run in a Docker image

```
cd path/to/blog
docker run --rm -v $(pwd):/srv/jekyll -it -p 4000:4000 jekyll/jekyll:latest jekyll serve
```
