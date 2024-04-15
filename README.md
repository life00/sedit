# SEdit

```
 ▄▄▄▄  ▄▄▄▄▄▄     █    ▀      ▄
█▀   ▀ █       ▄▄▄█  ▄▄▄    ▄▄█▄▄
▀█▄▄▄  █▄▄▄▄▄ █▀ ▀█    █      █
    ▀█ █      █   █    █      █
▀▄▄▄█▀ █▄▄▄▄▄ ▀█▄██  ▄▄█▄▄    ▀▄▄
```

[^1]

[^1]: Generated using [TOIlet utility](http://caca.zoy.org/wiki/toilet): `echo "SEdit" | toilet -f mono9`

A minimal POSIX shell script that allows you to securely edit any file as root with your preferred unprivileged editor.

## Disclaimer

**USE THIS PROGRAM AT YOUR OWN RISK.** I **DO NOT** provide any kind of guarantees as stated in the license (see [`LICENSE`](./LICENSE) file).

## Security considerations

It is important to mention that I personally do not use this program because I don't frequently edit root owned files, and generally I have some doubts about the fundamental concept. I believe that the fundamental idea of editing a root owned file with an unprivileged editor does not really sound very secure, and it opens up a lot of potential for exploitation. I have created this project only as a proof of concept, however it does not mean it is unmaintained or insecure. With this project I am trying to achieve the most secure implementation of this idea, however you still should consider the risk and the fact that **I provide no guarantees**. In case you believe there is an issue with the code do not hesitate to contact me (see my [profile](https://github.com/life00) for contact info).

## Design

The [original project](https://github.com/TinfoilSubmarine/doasedit) on which this one is based on (see [Credits](#credits)) has the similar goal of providing an ability to edit root owned files with your preferred unprivileged editor, however the main difference between projects is in what environment the program runs.

A while back I faced a problem where I would prefer to be able to edit root owned files with my customized editor, however there was no simple way and I was also using [`doas`](https://github.com/slicer69/doas). I knew that `sudoedit` functionality existed in [`sudo`](https://github.com/sudo-project/sudo) and so I tried to search a similar implementation, but for `doas`. I found many good projects, however there was one design issue I was not satisfied with, which is the fact that `doas` is triggered multiple times in the same environment with the unprivileged editor. Considering that for many users their unprivileged custom editor is a huge program with a risk for malicious software to compromise it without detection, I believe that it should be properly isolated and prevented from easily gaining root access by exploiting the weaknesses in the `sudoedit`-like programs (i.e. there is an assumption that the unprivileged editor is compromised and should not be trusted).

In this project I have addressed several such issues by having a different runtime environment. Instead of the program running as unprivileged user and then occasionally asking for a password and launching the editor, the program runs fully as root except the unprivileged editor for which it creates an unprivileged environment using `su`. Other than providing more reasonable security, it also provides more portability given that it is not necessary to have `doas` or `sudo` installed on the system to use this program. Furthermore, thanks to the minimal and concise code of the [original project](https://github.com/TinfoilSubmarine/doasedit) it is possible to use this program on any POSIX-compliant system.

In addition to that, considering the security assumption regarding the unprivileged editor it was necessary to also provide validation that the saved root owned file is not tampered with anything. I have approached this problem by including a brief prompt with a diff output if a user wants to apply the changes. Even though this may not be the cleanest solution, I believe it provides reasonable validation.

## Usage

Before using the program you are strongly encouraged to read the documentation (this readme) and the [code itself](./sedit) to fully understand how it works. After doing so you may proceed with the following installation steps:

1. Clone the repository into your current working directory: `git clone https://github.com/life00/sedit`
2. Copy the executable to `/usr/sbin` or `/usr/local/sbin`: `cp ./sedit/sedit /usr/local/sbin/`
3. Configure the environment variables in the executable using your preferred editor: `vi /usr/local/sbin/sedit`
   - `user` - the unprivileged user the editor will run as
   - `EDITOR` - the editor that will be used to edit the file
     - if the environment variable is already set then it will use it

After successful installation you may test the program by running it on any file as follows:

```sh
sedit ./path/to/file
```

The program expects the user to provide only one argument as a valid path to an existing file. This means that you can only edit existing files using this program, otherwise you will have to first create it by simply `touch ./path/to/file` before opening it with `sedit`.

## Credits

I would like to thank the original author of [doasedit](https://github.com/TinfoilSubmarine/doasedit), [Joel Beckmeyer](https://github.com/TinfoilSubmarine) for creating such a concisely written minimal program on which code this project is based on. The [original project license](https://github.com/TinfoilSubmarine/doasedit/blob/main/LICENSE) is BSD 2-Clause License which is also used for this project and [included in the source code](./LICENSE).
