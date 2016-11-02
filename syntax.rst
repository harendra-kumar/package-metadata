A Common Package Metadata Spec Lanugage
---------------------------------------

If we are convinced that it will be beneficial to use a unified, modular and
extensible package metadata in a common syntactic language across all tools,
what would that language be?

A discussion on this topic happened on Haskell cafe mailing list. This document
tries to capture the main points raised in that discussion.

We have the following options for the language:

* Existing .cabal language (separate it - or at least modularize it - from
  the cabal package so that it can be maintained independently). Also keep the
  spec separate from the tool so that it can independently evolve.
* `YAML <http://yaml.org/spec/1.2/spec.html>`_ is a popular language for
  configuration, serialization purposes.
* `TOML <https://github.com/toml-lang/toml>`_
* A restricted subset of Haskell

quotes
^^^^^^

"This is a worthwhile goal IMHO, but we need to be more concrete, e.g. how can
repetitive stuff like the tons of almost-copy-n-paste in
https://github.com/haskell-opengl/GLUT/blob/master/GLUT.cabal be avoided? This
has nothing to do with syntax, more with abstraction facilities and semantics:
If we just switch to JSON or YAML, GLUT.cabal would as repetitive as before,
only in a different surface syntax." - Sven Panne

"Personally, I would e.g. like to see some abstraction facilities to avoid
repetition in .cabal files with lots of executables, but I don't care about the
concrete syntax (and Cabal's internal model/AST wouldn't be affected, either)."
- Sven Panne

Existing .cabal format
^^^^^^^^^^^^^^^^^^^^^^

Pros:

The favorable point of .cabal format is backward compatibility.  Also, it is
custom built so we can add whatever is needed.

"It's still quite possible that it's simply not worth it; the cons associated
with changing the buildfile format are pretty weighty after all, and if the
Cabal people say they can fix the known problems with that format, it's
probably a better idea to see what comes of that before pursuing alternate
formats." - Joachim Durchholz

Cons:

"cabal is ok, but very imperfect, I generally need to have a lot of
copy/paste, I need to change it very often while writing application with many
dependencies" - yogsototh

"There is one substantial disadvantage I'd point out to the Cabal file format
as it stands, and that's that it's pretty non-obvious how to parse it, so we
will always struggle to interact with it from automated tools, unless those
tools are also written in Haskell and can use the Cabal library.  That's a real
concern; pragmatic large-scale build environments are not tied to specific
languages, and include a variety of ad-hoc third-party tooling that needs to be
integrated, and Cabal remains opaque to them." - Chris Smith

The use of YAML in many new tools and even for generating .cabal files is an
indication of the problems (technical or practical) with using the cabal
format. For example, the following new tools use YAML for their configuration:

* `stack <http://www.haskellstack.org>`_
* `hpack <https://hackage.haskell.org/package/hpack-convert>`_
* `iridium <https://hackage.haskell.org/package/iridium>`_

Other drawbacks:

* Approachability: Some have pointed out that it is not practically
  approachable for easy extensions for new or experimental use cases because
  the ownership is tied to one particular tool ecosystem.

* Maintenance: Even if it becomes more approachable it will require significant
  effort (and associated latencies) to ensure that we do not create a mess by
  adding point features hastily for any use case that pops up, without giving
  enough thought to generality and good design.  The premise is that it is not
  easy to design a good general and concise language to express configurations.

* Verbose: It already has some serious drawbacks which have not been
  addressed for a long time. For example, it does not allow reuse, creating
  redundancy which is ironically against Haskell's basic conciseness and reuse
  principles.

Work in progress:
  Oleg is currently working on a new parser for cabal.config,
  cabal.project & ${pkg}.cabal grammar (NB: cabal already uses one
  standard unified syntax for all its configuration/description files)
  which lends itself better to provide equivalent of ghc-exactprint
  (i.e. perfect roundtripping, allowing for faithful refactoring
  tooling). Then 3rd parties can then use this new parser as a library.

YAML
^^^^

Pros:

* It is a general, data oriented specification language existing for more than
  a decade and quite popular. Implementations are readily available and stable.
  Therefore not much maintenance overhead on us.
* A wider programmer base familiar with the language, newcomers might possibly
  already be familiar with it and feel at home. Or if they learn it fresh then
  the knowledge can be reused elsewhere.
* "It does matter for people who already know JSON: They can skip over the
  config file syntax and dive right into the semantics.
  Given that a substantial fraction of programmers knows JSON, using that
  syntax would create a lower entry barrier.

  The same argument can be made for YAML." - Joachim Durchholz
* Bonus - Easily serializable
* Has been proven to express package metadata by hpack, stack and other tools.
* "The fact that both Yaml and JSON can be represented as Aeson Values would
  also make things (arguably) easier for tool writers." - Tobias Dammers
* "One major benefit of YAML that I haven't seen mentioned is that it could be
  used to replace the README.md file at the same time. Right now a package
  description consists of both .cabal and (optionally) Markdown. I suspect the
  latter language is actually harder for complete beginners." - Mario Blažević

Cons:

* Will have to manage the package spec change from .cabal to .yaml in the
  ecosystem
* Complex specification (not sure how that matters as long as we do not have to
  maintain it). The mitigation is to restrict the features to a simple subset.
  Is there an easy way to restrict the syntax?

Herbert Valerio Riedel's comments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

I'm not sure if this has been pointed out already, but beyond turning a
proper grammar into a stringly-typed one, shoehorning some features of
.cabal files into YAML syntax really appear like a case of the "Genius
Tailor"[1], e.g. consider the `hpack` example::

   when:
     - condition: flag(fast)
       then:
         ghc-options: -O2
       else:
         ghc-options: -O0

besides looking quite awkward IMHO (just as an exercise, try inserting a
nested if/then/else in that example above), the prospect that a standard
format like YAML would allow to reuse standard tooling/libraries for
YAML seems quite weak to me; if, for instance, you run the above through
a YAML pretty-printer, you easily end up with something like::

   when:
   - else:
       ghc-options: -O0
     then:
       ghc-options: -O2
     condition: flag(fast)

or any other ordering depending on how the keys are sorted/hashed.

Besides, many YAML (& JSON) parsers silently drop duplicate keys, so if
by accident you place a 2nd `else:` branch somewhere, you end up with an
ambiguous .yaml file which may either result in an error, in the first
key getting dropped (most likely variant), or in the 2nd key getting
dropped. Which one you get depends on the YAML parser implementation.

----

The if/then/else awkwardness is just one aspect I pointed out
explicitly. I hinted at other issues which result from first parsing
into an inappropriate data-model just for the sake of using YAML, and
then having to re-parse that interim lossy data-model for real into the
actual data-model we're interested in (and hoping we didn't loose some
of the essential information).

TOML
^^^^

It is a simpler specification and designed for configuration file use case.
Rust lang uses it for Cargo.

Cons:

* Specification is still evolving, not stable and might keep changing
* Implementation will have to be maintained as the spec changes
* Relatively new and we do not have much experience with it to know whether it
  will be sufficient for all use cases.
* "TOML is limited in its data types: numbers, dates, strings for primitives,
  arrays and string-to-object maps.
  I'd consider that too limited to ever become a universal configuration
  format." - Joachim Durchholz

Haskell Subset
^^^^^^^^^^^^^^

The idea is to use a subset of Haskell for specifying the configuration. A
Haskell DSL can be designed and used if needed. This is an investigative idea
as we do not yet know details about how to do it and what subset will useful,
what problems might arise etc.

A working example of a Haskell DSL to express cabal files is here:
* http://hackage.haskell.org/package/cartel

Pros:

* Single language for everything, no need to learn anything new.
* It is a full fledged language with full expressive power, enabling to achieve
  whatever we want.
* Can be used in a more or less declarative manner

Similar examples:

* Google Bazel uses python, sbt uses scala
* "Sbt seems to be doing rather well, using full Scala in configurations." -
  MigMit
* "To draw an analogy, JSON derives from JavaScript. Isn't this a precedent?" -
  Imants Cekusins

Other points:

* "If we have to express not just a package specification but a sophisticated
  build configuration, we need a real language. Expressing conditionals, reuse
  etc becomes a compromise in a purely declarative language." - Harendra Kumar

* "- JSON/YAML/TOML are simply not powerful enough to match all semantics we
  might need to configure a project. For example we might want to have Set
  instead of List for some properties. Or I don't know maybe ternary tree
  structures." - yogsototh

* "for interop with other packagers / builders, .hs compatible config content
  could be transformed / exported to other formats." - Imants Cekusins

Cons:

* Needs investigation, implementation effort and time to prove.
* Bootstrap problem: "If you can't start or modify a package without already
  knowing haskell, it is a huge barrier to entry.  I remember trying to get
  started in scala and having a lot of trouble with sbt because I didn't know
  their operators for lists and arrays or hash tables or whatever it is that
  they use in their files." - David McBride

  "That is because they committed to the sin of employing the whole of
  Scala for the thing.  Bad for them." - Kosyrev Serge

  "I'm unconvinced that this problem cannot be resolved within the subsetting
  approach." - Kosyrev Serge

  "Actually subsetting is making this worse: Things freshly learned for Haskell
  won't work in the config language, restrictions encountered in the config
  language will be unthinkingly transferred to Haskell.

  Having two subtly but fundamentally different languages is about the worst
  thing you can expose a learner to." - Joachim Durchholz

  "I agree.  This is exactly what I felt when I tried to use Fay language,
  which is a «proper subset» of Haskell (in the end I switched to GHCJS)." -
  Geraldus

  "Haskell is indeed unsuitable for describing the package configuration, IMO,
  but not because it's lazy. It's because it lacks any syntax for long and
  human-readable string literals (package description, anyone?). That also
  condemns every subset of Haskell." - Mario Blažević

* "Scala's build system lets you do very powerful things, but it also makes
  things unnecessarily complicated and mystifying for beginners. At my previous
  work where we used Scala extensively, there were many times where the team
  simply resorted to external tools because figuring out how to make some
  seemingly trivial change to an SBT module was too time consuming." - Chris
  Kahn
  "Let me guess (have no idea about sbt) -- unbridled Turing completeness?

  Declarativity is king for configuration, and Turing completeness ain't it --
  please, see my other mail about subsetting Haskell." - Kosyrev Serge
* Restricted subset creating confusion when working on config vs on the actual
  program.
* "The more power you put into the package file description, the harder it is
  for the surrounding ecosystem to reason about it.  So if you can execute
  arbitrary code in a new-gen cabal file, apart from the security aspects, it
  becomes difficult to be sure what is actually being specified, if you do not
  reproduce the original environment when evaluating the file." - Alan
  Zimmerman

  "They could use their own Prelude and not allow importing other modules." -
  Imants Cekusins

  "The worst of all, IMO, is that it makes reasoning about the
  configuration equivalent to the halting problem." - Kosyrev Serge

  "That's a solved problem: Generate an execution plan, which would need to be
  fully evaluated in Haskell; then execute it and don't feed anything back into
  it.  It's easy to reason about the plan in that scenario." - Joachim
  Durchholz

