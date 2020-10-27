TCSH
====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

::

   complete 'pkg_*' 'C,*,`ls /var/db/pkg`,'
   complete 'portmaster' 'C,*,`cut -d\| -f2 /usr/ports/INDEX-9|sed -E s-/usr/ports/--`,'

::

   # Yamagi's svn
   set svn_cmds=(add annotate blame cat checkout cleanup commit \
        copy delete diff export help import info list log merge mkdir move \
        patch pdel pedit pget plist praise propdel propedit propget proplist \
        propset pset remove rename resolved revert status stat switch \
        update upgrade)
   set svnadmin_cmds=(create deltify dump help hotcopy list-dblogs \
        list-unused-dblogs load lstxns recover rmtxns setlog verify)
   set svnsync_cmds=(init sync copy-revprobs info help)
   complete svn 'p/1/$svn_cmds/'
   complete svnadmin 'p/1/$svnadmin_cmds/'
   complete svnsync 'p/1/$svnsync_cmds/'

   # Yamagi's git
   set git_cmds=(add add--interactive am annotate apply archimport archive bisect \
       bisect--helper blame branch bundle cat-file check-attr check-ref-format \
       checkout checkout-index cherry cherry-pick citool clean clone commit commit-tree \
       config count-objects credential-cache credential-cache--daemon credential-store \
       cvsexportcommit cvsimport cvsserver daemon describe diff diff-files diff-files \
       diff-index diff-tree difftool difftool--helper fast-export fast-import fetch \
       fetch-pack filter-branch fmt-merge-msg for-each-ref format-patch fsck fsck-objects \
       gc get-tar-commit-id grep gui gui--askpass hash-object help http-backend imap-send \
       index-pack init init-db instaweb log lost-found ls-files ls-remote ls-tree mailinfo \
       mailsplit merge merge-base merge-file merge-index merge-octopus merge-one-file \
       merge-ours merge-recursive merge-resolve merge-subtree merge-tree mergetool mktag \
       mktree mv name-rev notes pack-objects pack-redundant pack-refs patch-id peek-remote \
       prune prune-packed pull push quiltimport read-tree rebase receive-pack reflog \
       relink remote remote-ext remote-fd remote-testgit repack replace repo-config \
       request-pull rerere reset rev-list rev-parse revert rm send-email send-pack \
       sh-i18n--envsubst shell shortlog show show-branch show-index show-ref stage \
       stash status stripspace submodule svn symbolic-ref tag tar-tree unpack-file \
       unpack-objects update-index update-ref update-server-info upload-archive \
       upload-pack var verify-pack verify-tag web--browse whatchanged write-tree)
   complete git 'p/1/$git_cmds/'

   # Kami's Mercurial commands and options
   alias _comp_hg_options hg -v help `echo \$COMMAND_LINE:s/hg //:as/\ /./:ar` \
       \|\& awk '/options:/,/\\\$/'
   complete 'hg' \
       'p,1,`hg help | awk \/^\ /\{print\ \$1\}`,' \
       'n,help,`hg help | awk \/^\ /\{print\ \$1\}`,' \
       'n,--rev,`hg log | awk \/^changeset:/\{print\ \$2\}`,' \
       'n,-r,`hg log | awk \/^changeset:/\{print\ \$2\}`,' \
       'C,-,`_comp_hg_options`,'

* :ref:`genindex`

Zuletzt geändert: |date|

