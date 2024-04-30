# Shiva and the tale of Hindu Elves, Part-1: How to hack PacMan with Shiva

## Introduction

Presenting the Shiva blog-post series part 1. "How to hack PacMan with Shiva". Before delving in... 
Shiva is a state-of-the-art binary patching solution for Linux, allowing developers to write patches
for native software in C without having to recompile the original source code. If you are not yet
familiar with Shiva check out the official Shiva GitHub [1] and the official website for Shiva [2].
You will find resources for the Shiva user manual, and the recent DEFCON talk on Shiva.

Shiva has many features and is capable of re-writing virtually any part of the code or data within
a program at runtime. These feats are quite awesome, and can only be properly explained through an
interesting story to captivate the audience... we all love PacMan and it just so happens to be a game
written for Linux in C++, therefore it compiles into an ELF binary and can easily be patched with various
game cheats.

In this blog-post we will demonstrate a bit of basic reverse engineering and C coding to create two
different patches for PacMan for gaining invincibility and things like that :)

## Download and install Pacman for X86_64 Linux

This blog-post assumes that you have already installed Shiva for x86_64, please read the Shiva install
instructions from the Shiva user manual [3] if you have not already.

$ git clone https://github.com/tkilminster/pacman
$ 



















[1] https://github.com/advanced-microcode-patching/shiva
[2] https://arcana-research.io/shiva


