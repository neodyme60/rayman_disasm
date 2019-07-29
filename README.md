# rayman_disasm

Reverse-engineering effort for the 1995 MS-DOS game “Rayman”

## Introduction

A long time ago, I've been trying to reverse-engineer Rayman, for the sake of my own curiosity.

After several years of working on and off on it, here is the basic to get you started with the unpacking and disassembly of this game.

## Pull request welcome!

This repo is accepting pull requests, hoping that people will propose C reimplementation of certain functions.

## Version used

The version analyzed here is the one sold on [Good Old Games](https://www.gog.com/game/rayman_forever), as it is the easiest available of the full game.

However, the method described here can be applied to the many demos you can download online for free:

- [Internet Archive's Rayman demo](https://archive.org/details/Rayman_1020)
- [Internet Archive's Rayman Gold demo](https://archive.org/details/RAYGOLD)
- [Rayman-Fanpage demo](http://www.rayman-fanpage.de/character1/download/raydemo.exe)
- [Demo included in the French magazine Génération4 (issue 83) in December 1995](https://www.abandonware-magazines.org/affiche_mag.php?mag=27&num=5126&images=oui)
- [Rolling demo included in the French magazine Joystick (issue 66) in December 1995](https://www.abandonware-magazines.org/affiche_mag.php?mag=30&num=2501&images=oui)

## Process

### Quick analysis

Like many video games of this era, Rayman is a 32-bit linear executable wrapped inside a 16-bit DOS extender.

```
$ sha256sum RAYMAN.EXE
599b1df2bda0aea8364161c0526d28c2ed70e881d57a4b52eebd7992ea009eb6  RAYMAN.EXE
```

A simple string analysis can show which extender has been used:

```
$ strings RAYMAN.EXE | grep -i extender
PMODE/W v1.21 DOS extender - Copyright 1995, Daredevil and Tran.
```

Rayman is therefore using `PMODE/W` which is [available for free online](http://www.sid6581.net/pmodew/) both as binary and source code.

### Decompressing the binary

Reading the documentation of `PMODE/W` shows that the binary is first compressed, then binded to the extender.

The first step is to decompress the binary using a tool name `PMWUNLIT`, made int 1997 by “David Lopez (Barcelona) - david@jet.es”.

Finding this tool online is easy, but I came across two version — one cracked by “The C00LMAN of MAN WHQ. [SHG] - c00lman@usa.net”, the other not (?) — that produces different results.

The cracked version `PMWUNLIC.EXE` will be used as a reference here.

```
$ sha256sum PMWUNLI*.EXE
acdf80567c12b1b9144b2a0d9c648572d93abcdf5e015b008b2386500a7f8408  PMWUNLIC.EXE
dcbdaf0665b4d252620e0ff786a05e7c0ca9d9ee42986fe1d942ef0359db68be  PMWUNLIT.EXE
```

The uncompressed binary is significantly bigger. `RAYUC.EXE` is obtained using the cracked version.

```
$ sha256sum RAYU*.EXE
ead9356d01de781b4fc9cd8c88c44f9e2ef38c8cdf0d86e29907023c7a7e684b  RAYUC.EXE
5fcd0342fef77c96bf6fb288b758e2aba541b73d7c3a0f90627d76553ad9bb62  RAYUT.EXE
```

### Unbinding the extender

Now that the binary has been uncompressed, it is already more usable into a disassembler or debugger, but it is possible to completely get rid of the extender.

Both [PMODE/W](http://www.sid6581.net/pmodew/) and [DOS32A](https://dos32a.narechk.net/index_en.html) provide tools that produce exactly the same results.

For PMODE/W:
```
PMWBIND.EXE /U /Dout.exe in.exe
```

For DOS32A:
```
SB.EXE /U /UNout.exe in.exe
```

Now we have a linear executable. `RAYLEC.EXE` is obtained using the cracked version.

```
$ sha256sum RAYLE*.EXE
ca21e594de2dbae66e3b2a2681bc0daf72c8f0ccced7dd9618d51c71b8b1b9c9  RAYLEC.EXE
1a0aa289d2badd1a54c6f7f98ebaf6e4653afa9d03d50c23117ffaef8589f436  RAYLET.EXE
```

```
$ radiff2 RAYLEC.EXE RAYLET.EXE
0x00030f7b 389017 => a45d18 0x00030f7b
0x00030f7f 389017 => a45d18 0x00030f7f
0x00031653 e85fe5 => 7c92e4 0x00031653
```

### Disassembling

Some people have already worked on the disassembly of DOS linear executables. The ones behind [the port of Syndicate Wars to Linux](http://swars.vexillium.org/) made a presentation named “[How to port a DOS game to modern systems](https://recon.cx/2010/slides/recon_swars.pdf)” at REcon in 2010.

For this purpose, they design their own tool [`swdisasm`](http://swars.vexillium.org/files/swdisasm-1.0.tar.bz2) which has been forked to [`le_disasm`](https://github.com/samunders-core/le_disasm).

While the two Rayman binary we have bare a small difference, the assembly output is identical.

## Cracked version analysis

On December 16th, 1995, the warez group “Hybrid” released a cracked version of Rayman, only 3 months and 15 days after its original release.

The cracker of the game, someone named “Replicator”, wrote the following about the copy-protection:

> The protection was a BITCH!
> These french guys has to be paranoid or something…
> It contained at least 10 CRC checks and 10 CD checks…
> We've been working on this puppy for the last five days…
> Even the sound configuration util was protected.
> Should be done now… 'nuff said!

Disassembling the cracked version of the game may be good way to evacuate the unnecessary code that is not needed to reverse engineer.

### Analysis

Using the same analysis method on the cracked verion gives the following informations:

* The cracked version uses the PMODE/W DOS extender *v1.16*
* The game is *uncompressed*
* The LE binary is *11732 bytes shorter* than the original game

```
$ sha256sum raycracked*
614dba5f117bc8caf7a31be9deca96100a28fe1535e6fd2df58fc62f5ed11649  raycracked.exe
ec236e7fb1b5b3f767d2f1a77b85eed572dde0fd5d4b1b4528fa29dba36db381  raycracked.le
```


