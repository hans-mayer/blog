---
layout: post
title:  NTP classic with .ntprc file
date:   2022-06-20 11:36:00 CET
categories:
---


There is one reason why I use still classic NTP. If you apply the following patch to ntpq/ntpq.c you can use a ~/.ntprc file


```
--- ntpq.c.orig	2020-06-23 11:17:48.000000000 +0200
+++ ntpq.c	2022-06-18 20:06:56.319061365 +0200
@@ -5,6 +5,7 @@
 #include <ctype.h>
 #include <signal.h>
 #include <setjmp.h>
+#include <sys/stat.h>
 #include <stddef.h>
 #include <stdio.h>
 #include <sys/types.h>
@@ -613,6 +614,92 @@
 		}
 	}

+        /* .ntprc has been parsed for options.  However, commands (except those
+         * that happen to be spelled the same as options) haven't been executed.
+         * Here, commands that aren't options are parsed from the init file(s)
+         * and executed.  The most useful commands are configuration commands
+         * such as timeout, version, authenticate yes, as well as keyid, keytype
+         * and passwd.  'passwd' is a special case - the intitfile must not be
+         * world or group accessible.  Generally, it doesn't make sense to
+         * include communicating commands in an initfile, as the host is not
+         * known.  However, including a 'host' command is possible.
+         */
+        {
+            const char *const *initdir;
+
+            for (initdir = ntpqOptions.papzHomeList; *initdir; initdir++) {
+                const char *le = *initdir;
+                char *dotfile = NULL;
+                FILE *initfile;
+
+                if( le[0] == '$' ) {
+                    char *env;
+                    if (isalpha(le[1]) && ((env = getenv(le+1)) != NULL)) {
+                        dotfile = malloc(strlen(env) + 1 + strlen(ntpqOptions.pzRcName) + 1);
+                        sprintf(dotfile, "%s/%s", env, ntpqOptions.pzRcName);
+                    } else if (le[1] == '$') {
+                        ptrdiff_t plen = strlen(ntpqOptions.pzProgPath) - strlen(ntpqOptions.pzProgName);
+
+                        dotfile = malloc(plen + strlen(le+2) + 1 + strlen(ntpqOptions.pzRcName) + 1 );
+                        sprintf(dotfile, (strlen(le+2)? "%.*s%s/%s": "%.*s%s%s"),
+                                (int)plen, ntpqOptions.pzProgPath, le+2, ntpqOptions.pzRcName);
+                    }
+                } else {
+                    dotfile = malloc(strlen(le) +1 + strlen(ntpqOptions.pzRcName) + 1);
+                    sprintf(dotfile, (strlen(le)? "%s/%s":"%s%s"), le, ntpqOptions.pzRcName);
+                }
+
+                if ((dotfile != NULL) && (initfile = fopen(dotfile,"r")) != NULL) {
+                    char cmdbuf[MAXLINE], *sp, *ep;
+                    struct stat dotstat;
+
+                    if (fstat(fileno(initfile), &dotstat) != 0) {
+                        perror(dotfile);
+                    } else if (S_ISREG(dotstat.st_mode)) {
+                        if (debug)
+                            fprintf(stderr,"Processing %s\n", dotfile);
+                        while (fgets(cmdbuf, sizeof(cmdbuf), initfile) != NULL) {
+                            ptrdiff_t len;
+                            int opt;
+
+                            sp = cmdbuf;
+                            while (ISSPACE(*sp))
+                                sp++;
+                            if (ISEOL(*sp) || *sp == '#')
+                                continue;
+
+                            for (ep = sp; !(ISEOL(*ep) || ISSPACE(*ep)); ep++)
+                                ;
+                            len = ep - sp;
+
+                            /* Check to see if this token is a option.
+                             * (If an option is spelled the same as a command that does
+                             * something else, the option wins.  But that's really bad
+                             * design...so it won't happen.)
+                             */
+                            for (opt = 0; opt < ntpqOptions.optCt; opt++ ) {
+                                if (strncmp(sp, ntpqOptions.pOptDesc[opt].pz_Name, len) == 0)
+                                    break;
+                            }
+                            if (opt < ntpqOptions.optCt)
+                                continue;
+
+                            if (strncmp(sp, "passwd", len) || !(dotstat.st_mode & (S_IRWXG | S_IRWXO))) {
+                                docmd(sp);
+                            } else {
+                                fprintf(stderr, "%s: %.*s not processed - file must not allow group or other access\n",
+                                        dotfile, (int)len, sp);
+                            }
+                        }
+                        if (debug)
+                            fprintf(stderr,"End of %s\n", dotfile);
+                    }
+                    fclose(initfile);
+                }
+                free(dotfile);
+            }
+        }
+
 	if (numcmds == 0 && interactive == 0
 	    && isatty(fileno(stdin)) && isatty(fileno(stderr))) {
 		interactive = 1;
```

A file ~/.ntprc has two lines with the words 'keyid' and 'passwd'

```
  keyid and the keyid value
  passwd and the password
```

And this makes the live easier, especially in the test phase, because now all commands can be used all the time without entering a password.
