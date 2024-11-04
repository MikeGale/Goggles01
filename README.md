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

    ! homepage: specifies a homepage URL displayed on your Goggle's profile.
    ! issues: specifies a URL where users can report issues for your Goggle.
    ! transferred_to: Allows to transfer ownership of a Goggle.
    ! avatar:  specifies a *valid* HEX color code for your Goggle.
    ! license: specifies the license of a Goggle's instructions.

Goggles are self-contained text files hosted in Github or  Gitlab. They contain instructions defining how the Brave Search chooses and ranks results.  They do this by imposing exclusions / discards, boosts and deboosts (downranks) to the native rankings produced by the search engine. They can target specific URL patterns  (and, eventually, website titles and other aspects of Web pages) and indicate how each ranking should be altered.

A Goggle file consists of instructions, one per line. Empty lines, or comments (starting with an exclamation mark: '!') are ignored. If there are invalid instructions, the Goggle will fail on submission, there will be feedback on the problem.  Failed Goggles will not be executed.

## Background[^1]
[^1]:  [Whitepaper on the "Goggles" idea](https://brave.com/static-assets/files/goggles.pdf)

There are many web pages and domains on the Internet, (more than 300 million domains are listed in some places [^2]).  Typical web search engines cannot handle all of this content, so they discard a lot.  This effectively censors what search engines deliver, often making the best content **for you** unreachable using them.  Goggles is an approach to give some control to the user in return for some preparation work up front.
[^2]:  [One count of Internet domain by top level domain](https://research.domaintools.com/statistics/tld-counts/)

Current search engines aim to deliver results to the user within about one second.  To do that they typically split the search into several phases.  The first "recall phase" uses fast and simple techniques to obtain a lot of matches.  There may be billions of them.  These are then pruned to deliver probably a few thousand. The phases that remove most of the initial candidates are often known as precision phases, they are increasingly sophisticated and costly to run. The final phase is "ranking" which puts the results into the order that the user sees.

Goggles are most powerful when they apply to many results, not few.

>Consider the Goggle "Filter out the results from the top 1000 domains on the internet", which could be an interesting way to explore the internet. Applying this on the final result set for most queries would lead to very few results, if any, due to the inherent bias in most search engines to surface content from popular domains. The rules defined by Goggles are better applied to the largest candidate-set possible, so that the intersection between candidates and rules to be applied is not empty. Only when that intersection is large enough, will the re-ranking introduced by Goggles be noticeable.

For that to work Googles must be deeply integrated into the search system. Such integration poses issues:

1.  Efficiency: applying the rules against all elements of the candidate set (typically URLs) has to be extremely fast.
2.  Independence: the host search engine needs to have control over their index.

Independence is available to search engines running their own index, like Google, Bing, Yandex, Baidu and Brave. Search engines that rely on external indices may not receive large enough candidate sets.  For example DuckDuckGo, Qwant and Ecosia, which rely on the Bing API, are limited to what the API gives them.

Integrating this enhanced capability into search engines is not a trivial task.

The current Goggles system (2024-11-04 14:22:51) selects by URL.  This could be extended to other sorts of selector, like the author of a web page.


## Size and limits of Goggles definition files:

- Maximum file size 2MB
- Maximum number of instructions 100 000
- Maximum "\*" or "^" in an instruction, 2 each

## Instructions within the file, patterns and sites

The simplest instruction is a plain-text pattern which can be found in URLs. The following would match any search result whose URL contains the pattern as-is:

    /this/is/a/pattern

It is also possible to use some limited "globbing" capabilities such as the '\*' character which will match zero, one or more characters (note: the number of wildcards allowed in a given instruction is limited):

    /this/is/\*/pattern

The special character '^' can be used to indicate that an URL delimiter such as '/' or end-of-url can be matched. More specifically, '^' is equivalent to the regexp: [^\w\d._%-]|$

This means that it can match either the end of the URL or any character that is not a letter, digit, dot, underscore, percent sign or dash. Here are a few examples of characters that are matched by '^': / (slash), = (equal), \[ (bracket), : (colon), etc.

The domain referred to will often be expressed in a site=<Domain> clause.

In practice, you will usually want to use '/' (or any other specific separator like '=' or '?') most of the time in your patterns, except at the end of a pattern in cases where you want to be a little bit more generic, and express that your pattern should be either matching at the end of the URL or be followed by a separator and then arbitrary URL components.

> For example, |https<nolink>://example.org^ will match: 'https<nolink>://example.org', 'https<nolink>://example.org/' or 'https<nolink>://example.org/path'; but it will *not* match 'https<nolink>://example.org.ac', which is also a valid domain name starting with 'https<nolink>://example.org'.
> 
> Another example, /foo.js^ will match: 'https<nolink>://example.org/foo.js', 'https<nolink>://example.org/foo.js?param=42', 'https<nolink>://example.org/foo.js/' but it will *not* match 'https<nolink>://example.org/foo.jsx' (because it is not followed by a separator).

Also note that only 2 carets are allowed in an instruction.

    /this/is/a/pattern^
    |https://example.org^
    /foo.js^

By default, a pattern can match anywhere in the URL, but there are specific characters which can be used to indicate prefix or suffix matches: we call them "anchors".

The '|' character can be used at the beginning or end of an instruction to indicate anchoring. The following instruction will match a prefix of the URL:

    |https://en.

The following will match a suffix of the URL:

    /some/path.html|
    |https://brave.com|

## Site and Actions

In addition, each instruction can specify a list of options, following the '$' character and separated by commas (','). Options can be used to more finely target specific search results, or to indicate how a matched result's ranking should be altered.

The most basic option is 'site=', which can be used to limit a instruction to a specific website, based on its domain. Options can be specified on their own (e.g. if you want to target any page of a site) or in conjunction with a pattern:

    $site=brave.com
    /blog/$site=brave.com


More options can be used to refine your targets. By default any instruction will apply to a URL, in future other aspects of a page may be added.  The following are mentioned in the Whitepaper:

    ! web3$inurl
    ! web3$intitle
    ! web3$indescription
    ! web3$incontent

Finally, you can specify an 'action'.  **These actions are the elements that change the search results you get.**  There are three of them: boost, downrank, and discard.  An instruction without an explicit action is a 'boost':

    /r/brave_browser/
    /r/brave_browser/$boost
    /r/brave_browser/$boost=2
    /r/brave_browser/$boost=3

The amount of a boost is an integer between 0 and 10.  They give some control of matches in the result order.

You can also downrank results:

    /r/google/$downrank
    /r/google/$downrank=2
    /r/google/$downrank=3

You can also discard results completely:

    $discard,site=idontwanttobepartoftheresults.com
    /this/is/spam/$discard

If several actions can apply to a particular URL only one is used, as follows:  a discard takes precedence, then the highest boost, then the lowest downrank.

## From the developers.

Individually, each instruction can either target a very narrow set of pages (or even a single page), or a wider range of them, to apply reranking to a bigger set of results. In combination, hundreds or more instructions can allow you to express complex reranking functions.

Although the Goggles language could express instructions to search through a small set of websites or act as a blocklist, Goggles really shine when used to express boosting and downranking across many domains and pages.

Goggles also impacts the behaviour of features such as Discussions and News or Videos clusters, giving you greater control on your search experience.

We are already using Goggles internally, and we can't wait to see what you will do with them!

To learn more about Goggles, we recommend that you read the "Learn by example" section of our "Getting Started" page here:

- https://github.com/brave/goggles-quickstart/blob/main/getting-started.md#learn-by-example
- https://github.com/brave/goggles-quickstart/blob/main/getting-started.md
- https://github.com/brave/goggles-quickstart/blob/main/goggles/quickstart.goggle

## Other requirements

To use Goggles effectively you need to maintain a list of favoured targets, and details of how much they are favoured.  Such a system could have information about how much a URL is favoured for different search purposes.  It might be able to construct parts of Googles programmatically.

The list should contain targets that you wish to avoid entirely.

Note:  Targets that you wish to avoid in one context, you may wish to see in another.  For example, known disinformation sources.

*(A target is the combination of the matching pattern, and the site.)*

An effective approach would be to find diligent and compatible people who also maintain target information and working with them, to reduce your personal load.

Where Goggles are on the radar of people who create Internet content, you may experience others attempting to influence their own boost factors and those of others.  This will generally be a form of "contamination" though it could be handled as in your system within a "perverted" list maintained to illustrate the impact of such influences.  You should declare such influence in any list you publish, both for yourself and anybody else using the Goggle.