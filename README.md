# Goggles01
Search Engine "Goggles" for the Brave search engine.

Online at:  https://github.com/MikeGale/Goggles01

Licence:  CC0-1.0 license

Brave googles are specifications that define how search is performed in the Brave engine.

They are simple text files.

The file is headed by:

! name: <Name>
! description: <Descrip>
! public: false
! author: <Name>

which are mandatory.

The heading can contain optional fields / metadata attributes:

! homepage — specifies a homepage URL displayed on your Goggle's profile.
! issues — specifies a URL where users can report issues for your Goggle.
! transferred_to — Allows to transfer ownership of a Goggle.
! avatar —  specifies a *valid* HEX color code for your Goggle.
! license — specifies the license of a Goggle's instructions.

! Goggles are self-contained text files hosted in Github or  Gitlab. They contain instructions defining how the Brave Search chooses and ranks results.  They do this by imposing exclusions / discards, boosts and deboosts (downranks) to the native rankings produced by the search engine. They can target specific URL patterns  (and, eventually, website titles and other aspects of Web pages) and indicate how each ranking should be altered.

A Goggle file consists of instructions, one per line. Empty lines, or comments (starting with an exclamation mark: '!') are ignored. If there are invalid instructions, the Goggle will fail on submission, there will be feedback on the problem.  Failed Goggles will not be executed.

Size and limits:
Maximum file size 2MB
Maximum number of instructions 100 000
Maximum "*" or "^" in an instruction, 2

Instructions within the file

The simplest instruction is a plain-text pattern which can be found in URLs. The following would match any search result whose URL contains the pattern as-is:
/this/is/a/pattern

It is also possible to use some limited "globbing" capabilities such as the '*' character which will match zero, one or more characters (note: the number of wildcards allowed in a given instruction is limited):
/this/is/*/pattern

The special character '^' can be used to indicate that an URL delimiter such as '/' or end-of-url can be matched. More specifically, '^' is equivalent to the regexp: [^\w\d._%-]|$
This means that it can match either the end of the URL or any character that is not a letter, digit, dot, underscore, percent sign or dash. Here are a few examples of characters that are matched by '^': / (slash), = (equal), [ (bracket), : (colon), etc.

The domain referred to will often be expressed in a site=<Domain> clause.

In practice, you will usually want to use '/' (or any other specific separator like '=' or '?') most of the time in your patterns, except at the end of a pattern in cases where you want to be a little bit more generic, and express that your pattern should be either matching at the end of the URL or be followed by a separator and then arbitrary URL components.

For example, |https://example.org^ will match: 'https://example.org', 'https://example.org/' or 'https://example.org/path'; but it will *not* match 'https://example.org.ac', which is also a valid domain name starting with 'https://example.org'.

Another example, /foo.js^ will match: 'https://example.org/foo.js', 'https://example.org/foo.js?param=42', 'https://example.org/foo.js/' but it will *not* match 'https://example.org/foo.jsx' (because it is not followed by a ! separator).

! Also note that the maximum number of carets allowed in a given instruction is limited.

/this/is/a/pattern^
|https://example.org^
/foo.js^

! By default, a pattern can match anywhere in the URL, but there are specific characters which can be used to indicate prefix or suffix matches: we call them "anchors".

The '|' character can be used at the beginning or end of an instruction to indicate anchoring. The following instruction will match a prefix of the URL:
|https://en.

The following will match a suffix of the URL:
/some/path.html|

! |https://brave.com|

Additionally, each instruction can specify a list of options, following the '$' character and separated by commas (','). Options can be used to more finely target specific search results, or to indicate how a matched result's ranking should be altered.

The most basic option is 'site=', which can be used to limit a instruction to a specific website, based on its domain. Options can be specified on their own (e.g. if you want to target any page of a site) or in conjunction with a pattern:
$site=brave.com
/blog/$site=brave.com


More options can be used to refine your targets. By default any instruction will apply to a URL, in future other aspects of a page can be defined:
! web3$inurl
! web3$intitle
! web3$indescription
! web3$incontent

Finally, you can specify an 'action', which indicates how the ranking of a matched result should be changed by your instruction. This is the mechanism through which you can customize the ranking of results to your liking. You can use one of three possible actions in your instructions, and by default, any instruction without an action will be considered as 'boost':
/r/brave_browser/
/r/brave_browser/$boost
/r/brave_browser/$boost=2
/r/brave_browser/$boost=3

The value associated with the option indicate the 'strength' with which you want to alter the ranking (note: currently a maximum of 10). It can be used to boost results differently, inside of the same Goggle (e.g. some results should be favoured more than others).

You can also downrank results:
/r/google/$downrank
/r/google/$downrank=2
/r/google/$downrank=3

You can discard results completely:
$discard,site=idontwanttobepartoftheresults.com
/this/is/spam/$discard

From the developers.

Individually, each instruction can either target a very narrow set of pages (or even a single page), or a wider range of them, to apply reranking to a bigger set of results. In combination, hundreds or more instructions can allow you to express complex reranking functions.

Although the Goggles language could express instructions to search through a small set of websites or act as a blocklist, Goggles really shine when used to express boosting and downranking across many domains and pages.

Goggles also impacts the behaviour of features such as Discussions and News or Videos clusters, giving you full control on your search experience.

We are already using Goggles internally, and we can't wait to see what you will do with them!

To learn more about Goggles, we recommend that you read the "Learn by example" section of our "Getting Started" page here: https://github.com/brave/goggles-quickstart/blob/main/getting-started.md#learn-by-example

https://github.com/brave/goggles-quickstart/blob/main/getting-started.md

https://github.com/brave/goggles-quickstart/blob/main/goggles/quickstart.goggle