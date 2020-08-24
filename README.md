Share Secrets
=============
A Python 3 wrapper for the [debian ssss package][1] implementation of [Shamir's Secret Sharing Scheme][2].

Secret Sharing
--------------
Securely sharing secrets has important implications in terms of disaster recovery and inheritance. Consider the following scenario:

A seed phrase for a hardware wallet that holds cryptocurrency assets is symmetrically encrypted with a strong password. The password is memorised by the wallet owner.

The encrypted seed phrase is safely backed up in multiple locations. Should the hardware wallet be damaged or lost, the seed phrase can be easily accessed and decrypted by the owner. The wallet can be re-generated and no funds are lost.

However - if the owner should die or suffer memory loss, the funds will be inaccessible. The encrypted seedphrase will be effectively locked forever without the password.

Sharing the password is obviously risky - it involves trusting that the holder of the password will not re-create the wallet and steal the funds.

Secure secret sharing solves this dilemma. The **secret** can be split into **shares** in such a way that each share on it's own contains no information that would allow the secret to be re-constituted. A _threshold_ number of shares are required for the secret to be re-constituted. 

This means that shares could be distributed amongst participants for whom collusion would be difficult (or ideally impossible).

The Current Project
-------------------
This project is a proof-of-concept that intends to work through the game-theoretic aspects of splitting and sharing secrets securely. It is intended to be run in an offline TAILS instance.

The secret is split into shares using the `ssss` utility - this is installed by default on Tails, and can be installed on Debian/Ubuntu based systems with `sudo apt install ssss`.

Running `ssss` on the command line outputs shares as a block. For example, to split the secret "password123" into 5 shares, requiring any 3 shares to rebuild the secret, wiht the prefix `csknk`:

```bash
ssss-split -t 3 -n 5 -w csknk
Generating shares using a (3,5) scheme with dynamic security level.

# The programme is interactive - prompts for input on stdin
Enter the secret, at most 128 ASCII characters: Using a 88 bit security level. 
csknk-1-098132bbf73104961a0860
csknk-2-39d0ea98be55c5fef54169
csknk-3-74bbbba1aeb8af8ce0a041
csknk-4-212fdc17b582c72abf11db
csknk-5-6c448d2ea56fad58aaf0e1
```

The block of 5 seemingly random hexadecimal numbers (each prefixed with the specified prefix and an index number) constitute the **shares**. Any three shares can be used to re-constitute the original secret:

```bash
ssss-combine -t 3
Enter 3 shares separated by newlines:
Share [1/3]: csknk-1-098132bbf73104961a0860
Share [2/3]: csknk-3-74bbbba1aeb8af8ce0a041
Share [3/3]: csknk-5-6c448d2ea56fad58aaf0e1
Resulting secret: password123
```
Generating the shares from the secret leaves you with a problem - anyone that gets hold of the threshold number of shares (in this case, three) they can rebuild the secret. As such, the secrets must be carefully kept separate and securely deleted once they have been distributed to their endpoints.

This project is an experiment in generating shares of a secret in such a way that they can be split and delivered safely.


Usage
-----
To split a secret, run `./split.py`. This runs an interactive terminal session that configures and runs `ssss`.

The output shares are stored as individual files, each with a suitable descriptive introduction. This eases the process of creating and distributing your secret shares.

You will be provided with an option to securely delete the share files as part of the splitting process - you should copy and distribute the files separately before shredding. Secure deletion uses the GNU `shred` utility. If you want to shred the share files at a later stage, use the command:

```
shred -vfzu filename.txt

# Alternatively, shred all files in a directory:
# MAKE SURE YOU'RE IN THE RIGHT DIRECTORY!
cd /path/to/shares
find . -type f -exec shred -vfzu {} +
```
If you're in a Tails session, you can use the "Wipe" utility in the Nautilus (GUI) file manager.

## Configuration
Save `sample-config.json` as `config.json` and enter your personal details.

Each share file will contain your name and an email address as specified in the config - this may make it easier to recover secret shares. The file also contains instructions for the share holder.

Rebuild the Secret from Fragments
----------------------------------
To rebuild the secret, run `ssss-combine` on the command line, specifying the required threshold level. At the interactive prompt, enter the required threshold number of the generated shares (in any order):

```
# If the threshold is 3
ssss-combine -t 3
```

Useful Resources
----------------
Generate printable password:

```sh
# Generate a pseudo random 44 printable character password
# Note that a 32 byte input resolves to 44 base64 encoded characters
head -c 32 /dev/random | base64

# Generate printable password, alnum + punctuation
tr -dc '[:alnum:][:punct:]' < /dev/urandom | head -c 32 ; echo
```

## References
>Shamir's Secret Sharing is defined with Lagrange's polynomials, precisely: there is a unique polynomial of degree at most t-1 which goes through t given points, and it is rebuilt as a linear combinations of Lagrange's polynomials. When the secret is shared, a random polynomial P of degree at most t-1 is generated, such that P(0) is the secret. Then user i receives share P(i). Any t shares are enough to rebuild P and recompute P(0)
>
> [Tom Leek in SO comment][12]

* [ssss][1] - C implementation, present by default in standard Tails installation
* [Shamir's Secret Sharing][2] - an overview, with sample Python code
* [GNU Shred][3] - securely delete files on a unix/linux system
* [Python gfshare package][4]
* [Description of python-gfshare][5]
* [libgfshare source package in Ubuntu][6] - C implementation
* [libgfshare man page][8]
* [Blockstack secret sharing][7] - python implementation
* [Interesting fork of libgfshare][9]
* [Secret Sharing Made Short][10](Hugo Krawczyk, 2001)
* [Issues with SSSS Package][11]
* [Tails home page][13]

[1]: http://point-at-infinity.org/ssss/
[2]: https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing
[3]: https://manpages.debian.org/jessie/coreutils/shred.1.en.html
[4]: https://lamby.github.io/python-gfshare/
[5]: https://chris-lamb.co.uk/projects/python-gfshare
[6]: https://launchpad.net/ubuntu/+source/libgfshare/1.0.5-3
[7]: https://github.com/blockstack/secret-sharing
[8]: http://manpages.ubuntu.com/manpages/xenial/en/man7/gfshare.7.html
[9]: https://github.com/jcushman/libgfshare
[10]: https://link.springer.com/chapter/10.1007%2F3-540-48329-2_12
[11]: https://security.stackexchange.com/a/83924/58780
[12]: https://security.stackexchange.com/a/49311/58780
[13]: https://tails.boum.org/
