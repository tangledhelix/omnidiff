# About Omnidiff

Omnidiff is a general purpose file monitor. It watches for changes, and
reports them via email (in universal-diff format).

Only files that Omnidiff knows about will be monitored. The configuration file
defaults to `/etc/omnidiff.conf`, but you can specify an alternate path at
run-time by passing it as an argument, e.g.

    omnidiff /usr/local/etc/omnidiff.conf

Get Omnidiff at: https://github.com/tangledhelix/omnidiff

# Requirements

At least Perl 5.6, and the `Text::Diff` module.

# License

This software is released under the Simplified BSD License. See the `LICENSE`
file for the full text.

# Installing Omnidiff

Generally you want something like this:

    cd && git clone https://github.com/tangledhelix/omnidiff.git
    cd ~/omnidiff
    cp bin/omnidiff* /usr/bin
    cp etc/omnidiff.conf.sample /etc/omnidiff.conf

Then edit `/etc/omnidiff.conf` to your liking.

If upgrading, make sure you don't overwrite your existing `omnidiff.conf`.

If you don't have root access, see below for installing as an
unprivileged user.

# Email parameters

By default, email will be sent from `root` to `root`. You can change those
values in the config file by setting `email_from` and `email_to` to other
email addresses.

By default, the email will be sent using `/usr/sbin/sendmail`. You can change
that by setting `email_program` in the config file.

You can customize the subject by setting `email_subject` in the config file.
If you insert the token `%d`, it will be replaced by the number of files which
this email shows changes for. The default subject line is:

    [omnidiff] %d file(s) changed

# Defining individual files to watch

Any line starting with the word `file` defines a file to watch for changes.
This *must* be a fully qualified path.

    file /etc/passwd
    file /etc/group

# Defining files using file globbing

Any line starting with the word `files` defines a glob pattern. You can use
wildcards or grouping patterns, as in these examples.

    # Match all files in a directory
    files /path/to/dir/*
    
    # Match all *.cfg files
    files /path/to/dir/*.cfg
    
    # Match *.c and *.h files
    files /path/to/dir/*.[ch]
    
    # Match *.cfg and *.ini files
    files /path/to/dir/*.{cfg,ini}
    
    # Match one character position with '?'
    # This would match dns21, dns22, dns23, ...
    files /path/to/dir/dns2?.conf

Omnidiff automatically ignores certain files such as Emacs and Vim swap/backup
files, and RCS/CVS archive files.

    *~  *,v  #*  .*.swp

Lastly, globbing ignores directories, because we can't diff them. Or at least,
omnidiff doesn't know how to diff a directory. This automatically excludes some
things we don't need, such as `RCS/`, `CVS/`, `.svn/`, and `.git/`.

# Defining filters

Any line starting with the word `filter` defines a filter pattern to use for
the most recently defined `file`. A filter pattern is typically used to
obscure sensitive data in the diff output; since it travels by email, we don't
want to include passwords, for example.

Filters can use any valid Perl regex syntax. See [perlre(1)][perlre] and
[perlreref(1)][perlreref] for more information.

[perlre]: http://perldoc.perl.org/perlre.html
[perlreref]: http://perldoc.perl.org/perlreref.html

    file /etc/shadow
    filter s/^([^:]+):[^:]+:/$1:******:/gm

The above example will watch the shadow file, but filter out the encrypted
password before sending the email.

Note that you should use the `/gm` modifier on most filters.  Omnidiff reads
entire diff files into a string value, so you need `/m` to work correctly in
that context.  For the same reason `/g` is needed; otherwise only the first
match is changed! I have not enforced `/gm` though, because it's possible
there are edge cases where you'd want the other behavior.

# Testing filters

You may find `omnidiff-filter-test` to be helpful in writing filters. Run it
without arguments for help on using it.

Testing a filter is always a good idea. Otherwise, there's no way to really
know if your filter code works.

# Cached files

Omnidiff uses a cache directory as a holding pen for working files. For each
file Omnidiff watches, there will be three files in the cache directory.

1. A copy of the current version of the file
2. A copy of the file as it appeared before the most recent change
3. A diff of the changes between the above two files

(Files that have been configured in Omnidiff, but which have not yet been
changed, will have only the first file.)

Generally these files are only there for Omnidiff to use. But from time to
time, it is handy to have the files available for review.

By default, Omnidiff uses `/var/cache/omnidiff` for this purpose, but you can
customize the path by setting `cache_dir` in the config file.

For files with a filter defined, Omnidiff only applies the filter to the email
it sends. The cached diff file contains the raw content of the diff. So in
cases where a diff email comes through and the filter obscures information
that you need to see, you can go to the diff file to review it. However, only
one diff is kept; if another change is detected in a subsequent run, the
previous diff is lost.

# Running via cron

Most likely you will want to run Omnidiff via cron so it can watch files for
you. Assuming you are using the default config file path, all you need to do
is call omnidiff.

    */5 * * * * /usr/bin/omnidiff

It could be you want mutiple config files to watch things at different
time intervals. That's easy too.

    */5 * * * * /usr/bin/omnidiff /etc/omnidiff-critical.conf
    0   * * * * /usr/bin/omnidiff /etc/omnidiff-hourly.conf
    0   0 * * * /usr/bin/omnidiff /etc/omnidiff-daily.conf

# Running as an unprivileged user

The examples above tend to assume you are a sysadmin running this script as
root, but you can just as easily run all of this out of your home directory
on a shared web host or similar.

    cd && git clone https://github.com/tangledhelix/omnidiff.git

Edit `$HOME/omnidiff/etc/omnidiff` to have...

    cache_dir /home/jdoe/omnidiff/cache
    email_from me@mydomain.com
    email_to me@mydomain.com

Run in your personal crontab:

    */5 * * * * /home/jdoe/omnidiff/bin/omnidiff /home/jdoe/omnidiff/etc/omnidiff.conf

