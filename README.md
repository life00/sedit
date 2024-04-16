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

A minimal POSIX shell script[^2] that allows you to securely edit any file as root with your preferred unprivileged editor.

[^2]: However it still depends on a non-POSIX program, see the end of [Design section](#design)

## Disclaimer

**USE THIS PROGRAM AT YOUR OWN RISK.** I **DO NOT** provide any kind of guarantees as stated in the license (see [`LICENSE`](./LICENSE) file).

## Security considerations

It is important to mention that I personally do not use this program because I don't frequently edit root owned files, and generally I have some doubts about the fundamental concept. I believe that the fundamental idea of editing a root owned file with an unprivileged editor does not really sound very secure, and it opens up a lot of potential for exploitation. I have created this project only as a proof of concept, however it does not mean it is unmaintained or insecure. With this project I am trying to achieve the most secure implementation of this idea, however you still should consider the risk and the fact that **I provide no guarantees**. In case you believe there is an issue with the code do not hesitate to contact me (see my [profile](https://github.com/life00) for contact info).

Since the initial release I have fixed `2` vulnerabilities in SEdit:

- `b2c3e97`
- `58c3ea1`

## Design

The [original project](https://github.com/TinfoilSubmarine/doasedit) on which this one is based on (see [Credits](#credits)) has the similar goal of providing an ability to edit root owned files with your preferred unprivileged editor, however the main difference between projects is in what environment the program runs.

A while back I faced a problem where I would prefer to be able to edit root owned files with my customized editor, however there was no simple way and I was also using [`doas`](https://github.com/slicer69/doas). I knew that `sudoedit` functionality existed in [`sudo`](https://github.com/sudo-project/sudo) and so I tried to search a similar implementation, but for `doas`. I found many good projects, however there was one design issue I was not satisfied with, which is the fact that `doas` is triggered multiple times in the same environment with the unprivileged editor. Considering that for many users their unprivileged custom editor is a huge program with a risk for malicious software to compromise it without detection, I believe that it should be properly isolated and prevented from easily gaining root access by exploiting the weaknesses in the `sudoedit`-like programs (i.e. there is an assumption that the unprivileged editor is compromised and should not be trusted).

In this project I have addressed several such issues by having a different runtime environment. Instead of the program running as unprivileged user and then occasionally asking for a password and launching the editor, the program runs fully as root except the unprivileged editor for which it creates an unprivileged environment (see [Securely launching the editor](#securely-launching-the-editor)). Other than providing more reasonable security, it also provides more portability given that it is not necessary to have `doas` or `sudo` installed on the system to use this program.

In addition to that, considering the security assumption regarding the unprivileged editor it was necessary to also provide validation that the saved root owned file is not tampered with anything. I have approached this problem by including a brief prompt with a diff output if a user wants to apply the changes. Even though this may not be the cleanest solution, I believe it provides reasonable validation.

Furthermore, thanks to the minimal, concise, and POSIX-compliant code of the [original project](https://github.com/TinfoilSubmarine/doasedit) SEdit code is also pretty minimal and very readable. Even though SEdit is also POSIX-compliant it is dependent on an external program to securely launch an unprivileged user session for the editor. This compromise has been made because there does not appear to be any other better and secure solution. This is further discussed in the following subsection.

### Securely launching the editor

By far the hardest challenge was to figure out how to securely launch and properly isolated the untrusted editor. From the beginning of development I was concerned about this process given that this is the main potential attack surface. After initial launch I received a report on matrix by user freedumbo (@freedumbo:arcticfoxes.net) who claimed that a threat actor who already compromised the editor program may easily gain root privileges by exploiting the way I use `su` to launch the editor. I was a bit surprised to say the least that it is possible to do it so easily. freedumbo has also provided me with a lot of resources explaining how it works and possible solutions to patch it.

The vulnerability is officially called _TTY pushback_ which is essentially about somehow escaping the unprivileged shell to the original root shell by exploiting the weaknesses in the environment. You may read more details about this in [these](https://www.halfdog.net/Security/2012/TtyPushbackPrivilegeEscalation/) [two](https://www.errno.fr/TTYPushback.html) articles. In the second article the solution presented is to use the `-P` argument for `su`, which solves this particular issue however it does solve the fundamental problem. In [one issue](https://github.com/systemd/systemd/issues/7451#issuecomment-346787237) poettering has suggested that the proper way of creating a clean environment is to use `machinectl shell` because the only right way a fully isolated session may be created is by the init system. I would probably agree with this statement given that is the core of the system that manages all the sessions, which also means that using `su` and similar to create an unprivileged session from privileged session may be fundamentally a bad idea.

After a bit more research I discovered that the same TTY pushback issue applies to [sudo](https://www.suse.com/support/kb/doc/?id=000021241) and [doas](https://github.com/Duncaen/OpenDoas/issues/106) where sudo [has a solution to this](https://github.com/sudo-project/sudo/issues/258) which is enabled by default while doas still does not.

I have addressed this problem by choosing the `machinectl` method as default, which means by default SEdit is dependent on systemd. I have also created an implementation for GNU `su` (but with `-P` argument) and `sudo` (it has this option enabled by default from 1.9.14 version, but users are still encouraged to enable `use_pty` option explicitly in config in case the default state changes in future versions), but this code is commented out by default. Users may choose which launching method they prefer, however I believe `machinectl` is the most secure method which is why it is the default. freedumbo has also suggested to use an ssh server to securely run the untrusted editor, however I believe this would require too much setup and don't think client OS's usually have an ssh server running (SEdit would probably be used more in client OS's), so I did not implement this option.

## Requirements

- POSIX-compliant system
- launching program options
  - `systemd-container`: `machinectl` (default and recommended)
  - GNU core utils: `su`
  - `sudo`

## Usage

Before using `sedit` you are strongly advised to read the documentation (this readme) and the [code itself](./sedit) to fully understand how it works. After doing so you may proceed with the following installation steps:

1. Clone the repository into your current working directory: `git clone https://github.com/life00/sedit`
2. Copy the executable to `/usr/sbin` or `/usr/local/sbin`: `cp ./sedit/sedit /usr/local/sbin/`
3. Configure the environment variables in the executable using your preferred editor: `vi /usr/local/sbin/sedit`
   - `user` - the unprivileged user the editor will run as
   - `EDITOR` - the editor that will be used to edit the file
     - if the environment variable is already set then it will use it
4. Choose your preferred launching program by (un)commenting sections of the code if necessary: `vi /usr/local/sbin/sedit`
   - `systemd-container`: `machinectl` (default and recommended)
   - GNU `su`
   - `sudo`

After successful installation you may test the program by running it on any file as follows:

```sh
sedit ./path/to/file
```

The program expects the user to provide only one argument as a valid path to an existing file. This means that you can only edit existing files using this program, otherwise you will have to first create it by simply `touch ./path/to/file` before opening it with `sedit`.

## Contribution

I think that this project is mostly complete because of its small scale, however if you believe there is something that should be added and / or fixed then feel free to create an issue or a pull request for me to review.

## Credits

I would like to thank the original author of [doasedit](https://github.com/TinfoilSubmarine/doasedit), [Joel Beckmeyer](https://github.com/TinfoilSubmarine) for creating such a concisely written minimal program on which code this project is based on. The [original project license](https://github.com/TinfoilSubmarine/doasedit/blob/main/LICENSE) is BSD 2-Clause License which is also used for this project and [included in the source code](./LICENSE).

Also, special thanks to all the volunteers and code reviewers that contributed to this project:

- @freedumbo:arcticfoxes.net (on matrix)
  - reported TTY pushback critical vulnerability: `b2c3e97`, `efa14e0`
  - reported race condition critical vulnerability: `58c3ea1`
