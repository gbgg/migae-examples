# gae

First App Engine exercise.

## Introduction

You can serve up your servlet using either jetty-runner.jar or the GAE
dev_appserver.  The difference is that with dev_appserver, all jar
dependencies must be in war/WEB-INF/lib, and all application code must
be either jarred up in war/WEB-INF/lib or in war/WEB-INF/classes.

If you put the app code in a jar, you lose interactive development.
You can get interactive development under dev_appserver, but at a cost:
you must put your source tree in war/WEB-INF/classes, and you must use
`:jar-exclusions` (see project.clj) to exclude the dynamic source from
your jar file.  Obviously this is not advisable, since the classes dir
and contents are dynamically created by `lein compile`.  So you would
not want to do this as a matter of course, but in exploring you might
as well give it a try - edit project.clj appropriately, copy
src/gae/user_impl.clj to war/WEB-INF/classes/gae, fire up
dev_appserver, edit war/WEB-INF/classes/gae/user_impl.clj, and refresh
the browser.  This just demonstrates that the Clojure runtime's code
loading and evaluation functionality continues to operate in GAE.

## Tasks

To run your servlet without interactive dev capabilities:

 1.  edit :aot in project.clj to aot compile everything.

 2.  copy dependency jars from ~/.m2/repository to war/WEB-INF/lib
 (later we will automate this).  See Note on Dependencies below.

 3.  disable the filter (comment it out in web.xml).  No point in
 reloading code.

 4.  run dev_appserver.sh as per official GAE instructions.

If everything goes well, you'll develop your code interactively using
jetty-runner.jar, and then system test it using dev_appserver.  It
will pass with flying colors.

In practice, you're likely to find problems with dev_appserver, so you
will inevitably end up switching between interactive and
non-interactive testing.  The goal is to minimize this and to make it
as easy and fast as possible.

Note that having jar dependencies in war/WEB-INF/lib has no effect
under jetty-runner.jar so you don't need to remove them once they're
installed.  So it really comes down to running migae-jetty.sh versus
running dev_appserver, and enabling and disabling the filter in
web.xml.

## A Note on Dependencies

You don't need to copy everything listed in :dependencies.  You will
always need the Clojure jar, since that contains the runtime.
Leiningen will arrange things so that the bits of code your
application relies on will be installed in war/WEB-INF/classes and the
app jarfile.  So, for example, you do not need to copy the compojure
jar into war/WEB-INF/lib, even though [compojure "1.1.5"] is included
in :dependencies.

However, Leiningen functionality depends on a set of naming
conventions.  Dependency names, like [compojure "1.1.5"], will
correspond in a systematic way with paths in ~/.m2/repository and with
classnames.  So when it compiles something that uses compojure,
Leiningen can find the jar in the repository, extract the source code
(try using `jar tf` to inspect the contents of the compojure jarfile
in the repo), compile it, and put the result in
war/WEB-INF/classes/compojure.  Then `lein jar` will put those class
files in your application jar.  The (only?) exception to this is the
Clojure runtime itself.

So long as maven packages follow the naming conventions, all will be
copacetic.  But sometimes that is not the case.  This project has an
example.  It indirectly uses ring-codec, which in turn depends on
org.apache.commons.codec.binary.Base64.  The problem is that that
class object is not stored under
~/.m2/repository/org/apache/commons/codec ... as Leiningen expects, so
it cannot find it.  At runtime, you will get a class not found error.
As it happens, this code is packaged in
`~/.m2/repository/commons-codec/commons-codec/1.7/commons-codec-1.7.jar`

Now there is probably a way to tell Leiningen to add that path to its
search path for `lein jar`, but I don't know what it is.  So for now,
the way to fix this sort of problem is simply to copy the needed jar
to war/WEB-INF/lib.

(NB: migae-jetty.sh uses the --lib option to tell jetty where to find
the jar, so it will work if it isn't in war/WEB-INF/lib.  But
dev_appserver won't; there is no way to tell the GAE dev server to look
outside of war/WEB-INF for classfiles.)

## License

Copyright © 2013 FIXME

Distributed under the Eclipse Public License, the same as Clojure.