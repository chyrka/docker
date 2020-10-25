This pam module provides the authentication service using an own set of user/password pairs.

FYI, https://github.com/tiwe-de/libpam-pwdfile

CONFIGURATION
=============

simple PAM config
-----------------

Just add/change the config file for service to contain the line:

auth		required	pam_pwdfile.so pwdfile=/path/to/passwd_file

If your service does more with PAM than auth there will be a fallback to the service "other".
If that is not what you want, you can use pam_permit.so or pam_deny.so for that:

account		required	pam_permit.so
session		required	pam_permit.so
password	required	pam_deny.so


options
-------

* pwdfile=<file>
* debug: produce a bit of debug output
* nodelay: don't tell the PAM stack to cause a delay on auth failure
* flock: use a shared (read) advisory lock on pwdfile, you should better move new versions into place instead
* legacy_crypt: see section LEGACY CRYPT


PASSWORD FILE
=============

The password file basically looks like passwd(5): one line for each user with two or more colon-separated fields.
First field contains the username, the second the crypt()ed password.
Other fields are optional.

crypt()ed passwords in various formats can be generated with mkpasswd from the whois package.


LEGACY CRYPT
============

There are two crypt types that are disabled by default: bigcrypt and broken md5_crypt.
They are disabled because they use static buffers which is bad when doing PAM authentication using this module in a multithreaded server.
All the other crypt types are checked via the systems crypt_r function if available, else with the normal crypt function and the same static-buffer-problem.

bigcrypt was used on DEC systems to allow for longer passwords.
You can check if your passwd file contains any of these with `cut -d: -f2 passwd-file | egrep '^[^$].{13}'`.

Broken md5_crypt is a speciality of big-endian systems.
An early implementation of md5_crypt got the byte order wrong here and produced different crypt outputs.
You might have some of these crypt hashes in your passwd file only if you created them on a big-endian system.
If an md5_crypt hash also worked on a little-endian system (up to and including libpam-pwdfile 0.99) it isn't broken md5_crypt.
