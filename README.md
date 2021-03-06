Automated encryption and decryption in Git repositories
=======================================================

It is possible to use automated encryption and decryption in Git repositories
because of the filter drivers that consist of a clean command and a smudge
command. Where the clean "command is used to convert the contents of worktree
file[s] upon checkin" in a similar way to how the smudge "command is fed the
blob object from its standard input, and its standard output is used to update
the worktree file." (The quotes are from gitattributes(5).)

Working with OpenSSL symmetric-key encrypted files in a repository
------------------------------------------------------------------

**1)** If one does not know the password to the encrypted files, the workflow is
the same as always: work on the unencrypted files and commit the changes, etc.

**2)** This is not always the case, though. If you are one of the lucky ones
and know the password and the cipher, and want to take part of the encrypted
content (here ending with the file extension ".senc" (symmetric-key
encrypted)), the following commands will do the job:

	git clone git://example.com/repository.git
	cd repository
	git config filter.crypt_sym.clean "openssl CIPHER -a -nosalt -pass pass:PASSWORD"
	git config filter.crypt_sym.smudge "openssl CIPHER -a -nosalt -d -pass pass:PASSWORD"
	git config diff.crypt_sym.command PATH_TO_crypt_diff
	echo '*.senc filter=crypt_sym diff=crypt_sym' >> .git/info/attributes

Then finally checkout the files that you want to decrypt:

	git checkout -- FILE.senc FILE2.senc

and work on them as usual.

**Notice** that you must replace CIPHER with the actual cipher that is used, in
this repository "example.senc" is encrypted using aes-256-cbc. The same is with
the PASSWORD, where it is "writecode" in this case. Also PATH\_TO\_crypt\_diff
must be replaced with the path to the shell script crypt\_diff, that is located
in this repository, in order to be able to view plaintext diffs between the
encrypted blob files and the plaintext worktree files.

**3)** To encrypt new files, all you have to do is to create them with the correct
file extension (if not a sole wildcard is used instead of "*.senc" in
.git/info/attributes) and later run `git add`. The same procedure is also used
when initializing a new repository, except that `git init` should be run before
the `git add` and that the two first steps are also skipped (`git clone` and
`cd`) in the process above.

Working with GnuPG public-key encrypted files in a repository
-------------------------------------------------------------

**1)** Same as in the previous subsection.

**2)** Same as in the previous subsection, except that the lucky one, in this
case, is having the private key corresponding to the public key NAME (remember to
change it below). (**Notice** that the "-r" option can be given multiple times,
which leads to broadcast encryption.) Changes made, in the above instructions,
are presented below.

	git config filter.crypt_pub.clean "gpg -ea -q --batch --no-tty -r NAME"
	git config filter.crypt_pub.smudge "gpg -d -q --batch --no-tty"
	git config diff.crypt_pub.command PATH_TO_crypt_diff
	echo '*.penc filter=crypt_pub diff=crypt_pub' >> .git/info/attributes

It is recommended to use a specific key pair only for this very repository and
to make sure that it is passwordless as the process is supposed to be
automatic.

**3)** Same as in the previous subsection, except that the file extension for
the encrypted files, in this example, is ".penc" (public-key encrypted) instead
of ".senc".

**CAVEAT**: Two GnuPG public-key encrypted versions of the same file are
different (because of the randomly-generated session keys), which leads to the
fact that Git will track these sort of "unreal changes" that usually occur when
one reverts a change. This is not the case with OpenSSL symmetric-key
encryption when salt is disabled.

See also
--------

[gpg(1)][2], [openssl(1)][3], [gitattributes(5)][4]

[1]: http://stackoverflow.com/questions/1557183/is-it-possible-to-include-a-file-in-your-gitconfig/1558141#1558141
[2]: http://www.gnupg.org/documentation/manpage.en.html
[3]: http://www.openbsd.org/cgi-bin/man.cgi?query=openssl&apropos=0&sektion=1&manpath=OpenBSD+Current&arch=i386&format=html
[4]: http://www.kernel.org/pub/software/scm/git/docs/gitattributes.html
