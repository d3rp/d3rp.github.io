# Hugo Pages for a Blog

## Development

The repo uses submodules. These should work with:

    git clone --recurse-submodules ...

If you have a cloned local version, cd into the directory and try:

    git submodule init
    git submodule update --recursive

TBH I'm still figuring these out.

### Files

The most relevant files are config.toml for hugo configuration, ./blog/\* for the actual content and .gitlab-ci.yaml for the gitlab pages deployment. The themes can be tweaked via ./layouts - it's better to leave the themes subfolder alone if deploying, as gitlab will override them by using git submodules. .gitmodules defines those themes to be used.

### Drafts

Hugo has drafts and published posts. These are distinguished in the header of the blog post.

    ---
    title: "Exit Bash"
    date: 2018-05-15T17:58:22+03:00
    draft: false
    ---

### Creating a new blog post

One can use several different markup formats (.md, .rst)

    hugo new blog/foobar.rst

This will basically create a new file in ./blog/foobar.rst with the aforementioned header.

### Live Rendering Locally

Final version can be seen via localhost:

    hugo server

.. with drafts:

    hugo server -D

## Deployment

Push to repo and the pipelines should trigger automatically

Also as an idea..

https://sadanand-singh.github.io/posts/nikola2hugo/

## Themes

Themes can be tested locally by cloning to themes/ and setting the name in config.toml. To make the theme deployable, it needs to be included in .gitmodules

Themes checked so far as plausible options:

* base16
* cocoa
* kiss

More here: https://themes.gohugo.io/

Usually things to keep in mind is how the code snippets are displayed, does it require lots of extra pages, is disqus available as an option..
