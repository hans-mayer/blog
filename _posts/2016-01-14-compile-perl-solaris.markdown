---
layout: post
title:  Compile perl 5.22 on Solaris 11
date:   2016-01-14 10:19:00 CET
categories: software perl 
---

Remove the original symbolic link 

<pre>
rm /usr/bin/perl
</pre>

Download the TAR archive and extract it. 

<pre>
gtar -xzvf perl-5.22.1.tar.gz
cd perl-5.22.1
export CC=gcc
export MAKE=gmake 
./Configure -des -Dprefix=/usr/perl5/5.22 
gmake
</pre>

As superuser "root"

<pre>
gmake install
rm /usr/bin/perl 
ln -s /usr/perl5/5.22/bin/perl5.22.1 /usr/bin/perl 

PATH=/usr/perl5/5.22/bin:$PATH
cpan
cpan[1]> make
q

perl -MCPAN -e 'install Some::Perl::Packages'
</pre>

