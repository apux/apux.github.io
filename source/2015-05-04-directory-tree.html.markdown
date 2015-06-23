---
title: Directory tree
date: 2015-05-04
tags: [tree]
---

For [my previous post](/2015/04/13/directory-structure-conventions-in-rspec-and-minitest.html), I have to show my projects' directory tree. Originally, I had copied-pasted it from my vim. However, there is a command for that task: `tree`. I didn't have it installed so I had to do it. I'm using `pacman` but you can use `yum`, `apt-get` or the equivalent one for your distro.

    sudo pacman -S tree

It is possible to run `tree` just by executing it.

```
[fizz_buzz_minitest]$ tree
.
├── lib
│   └── fizz_buzz.rb
└── test
    └── fizz_buzz_test.rb

2 directories, 2 files
```

this will recursively show the current directory structure, including files. If you want to show directories only, then use `-d` option.

```
[fizz_buzz_minitest]$ tree -d
.
├── lib
└── test

2 directories
```

It is possible to show the structure of a specific directory:

```
[fizz_buzz_minitest]$ tree /etc/X11

/etc/X11
├── xinit
│   ├── xinitrc
│   ├── xinitrc.d
│   │   └── 30-dbus
│   └── xserverrc
└── xorg.conf.d
    └── 10-keyboard.conf

3 directories, 4 files
```

If we want to see hidden files:

```
[fizz_buzz_minitest]$ tree -a
.
├── .git
│   ├── branches
│   ├── COMMIT_EDITMSG
│   ├── config
│   ├── description
│   ├── HEAD
│   ├── hooks
│   │   ├── applypatch-msg.sample
│   │   ├── commit-msg.sample
│   │   ├── post-update.sample
│   │   ├── pre-applypatch.sample
│   │   ├── pre-commit.sample
│   │   ├── prepare-commit-msg.sample
│   │   ├── pre-push.sample
│   │   ├── pre-rebase.sample
│   │   └── update.sample
│   ├── index
│   ├── info
│   │   └── exclude
│   ├── logs
│   │   ├── HEAD
│   │   └── refs
│   │       ├── heads
│   │       │   ├── master
│   │       │   └── nested
│   │       └── remotes
│   │           └── origin
│   │               ├── master
│   │               └── nested
│   ├── objects
│   │   ├── 25
│   │   │   └── 07828dee3799f528010c9b30a23fbe5f183f54
│   │   ├── 66
│   │   │   └── e163dc71cbbadc74435427b77ba5668b5326e6
│   │   ├── 69
│   │   │   └── 907733efe7e40b756fd5a6338fc746235d9912
│   │   ├── 7d
│   │   │   └── 3e2d67baa753b84ab89b443c81b2d518911ad5
│   │   ├── 9f
│   │   │   └── b847453d0407123ffff4c120fba4b65777c0bb
│   │   ├── b7
│   │   │   └── 10d9b96d88636105f08988fdf2ce9db09aa09b
│   │   ├── c8
│   │   │   └── 5b809b75fdce3cc4575853d6d19651de0273cd
│   │   ├── ca
│   │   │   └── 1320c9f7d74bc85b8764cf7e31dc916e6fa41b
│   │   ├── da
│   │   │   └── 148f531bc6252d23d325dd6c972fb67b8dae55
│   │   ├── ee
│   │   │   └── f295927a89e8217970deb8a2040a8d2a9af574
│   │   ├── f8
│   │   │   └── 7d0c12f09ba71821e8b4478ff2bc12ad5db570
│   │   ├── fc
│   │   │   └── a429691bead1ee677ae549a7dbefa30a5f2841
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       │   ├── master
│       │   └── nested
│       ├── remotes
│       │   └── origin
│       │       ├── master
│       │       └── nested
│       └── tags
├── lib
│   └── fizz_buzz.rb
└── test
    └── fizz_buzz_test.rb

31 directories, 38 files
```

Wow! those were many files for one basic project. The `.git` hidden directory has a lot of nested files and directories. We can specify how deep `tree` should display with `-L number`, where `number` is the level of nested directories/files to show (in our case, 2).

```
[fizz_buzz_minitest]$ tree -a -L 2
.
├── .git
│   ├── branches
│   ├── COMMIT_EDITMSG
│   ├── config
│   ├── description
│   ├── HEAD
│   ├── hooks
│   ├── index
│   ├── info
│   ├── logs
│   ├── objects
│   └── refs
├── lib
│   └── fizz_buzz.rb
└── test
    └── fizz_buzz_test.rb

9 directories, 7 files
```

It is possible to have a colored output with the `-C` option. This shows directories in blue, executable files in green, etc.

`tree` has a lot of options. I showed just a few of them, but you can see all these options by `man`.

    man tree

or

    tree --help
