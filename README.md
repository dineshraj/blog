# Blog using Jekyll

## Install

Instal RVM (use the gpg command from https://rvm.io/rvm/install first):
`$ \curl -sSL https://get.rvm.io | bash -s stable --ruby`
`$ export GEM_HOME="$HOME/.gem"`  (add to .zshrc or .bashrc)
`$ gem install jekyll bundler`
`$ bundle install`

## Usage

`bundle exec jekyll server`

## Alternatively, run in a Docker image

`docker run --rm -v `pwd`:/srv/jekyll -it -p 4000:4000 jekyll/jekyll:latest jekyll serve`
