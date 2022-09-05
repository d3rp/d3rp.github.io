---
title: "Vim With Cpp and Rust"
date: 2021-01-30T20:36:42+02:00
draft: true
---

vim plugins:
    dein - plugin installer
    deoplete - language completion (from the maker of dein)
    vim-syntastic - error reporting a.k.a linting
    easy-tags - (Wraul's fork) Tag completion and jumping to definitions
    tagbar - structure view of the file
    
    C++:
        deoplete-clang - clang integration for deoplete (c++)
        derekwyatt/vim-fswitch - switching between header and implementation file
        
    Rust:
        LanguageClient - LSP client for reference documentation
        rust.vim - rust syntax and completion

linux packages:
    excuberant-ctags - tags a.k.a. symbols for completion

deoplete should do autocomplete without having to trigger it manually and with
ctags you should get those project wide symbols included.

LSP is a protocol, in which the compiler and some other magic is used to verify
and thus suggest things in real time. The plugin hooks vim as a client to a
running LSP server.

YCM gave me so many headaches with updates and some other issues that I started
looking for alternatives. I replaced it with deoplete.

links
rust
rust.vim, syntastic and tagbar - https://github.com/rust-lang/rust.vim

c++
deoplete-clang - https://github.com/deoplete-plugins/deoplete-clang
