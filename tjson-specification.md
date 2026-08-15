# Text Json (TJSON) Specification v0.5.2

Created by R.F. Anthracite rfa@rfanth.com

TJSON specification (stands for Text JSON) ([textjson.com](https://textjson.com))

---

## GOALS

The world is now flooded with tree type data, often sent through JSON at some point, but the ability for humans to read JSON hasn't improved along with it.  Much of our tree data already contains mostly human readable things, often inputted by humans, with keys chosen for comprehensibility by programmers, but the deep tree formats preferred by computers make it really hard to read.  The outputs take up miles of vertical space for very little actual information, and it's about time someone did something about it.  As such, I've developed a JSON format for humans, called TJSON (short for Text Json), a round trippable to JSON format for humans that can be embedded in text.

Humans read from screens in squares.  Computers read character by character, ignoring linefeeds, making JSON look just fine, but any attempt to turn JSON into normal text generally looks like a thin line passing through the screen going down to infinity.  Pretty printing helps, but it can't address many of the core readability problems.  This is my attempt to fix that.

The primary goal is to be really easy for a human to read, while still being zero-ambiguity to a programmer and perfectly preserving all data on round trip that JSON implementations preserve, which is everything except for duplicate keys.  This requires vertical compactness, among other things.  A lot of JSON data almost works as regular human text, but is made difficult to read by the JSON format itself.  This format should look as little like code gibberish as possible while allowing a competent human to instantly understand the underlying data structures and types.  Tradeoffs are made here between having obvious types and not looking like code gibberish - and the tradeoff is basically if you know one simple rule (bare strings always have a leading space before them), you always know the type, and if you don't, it's understandable as text.  This both lets programmers know what they need to know while getting the best possible reading experience, and lets non-programmers just see language even if they barely know what a type is.

Humans usually will not edit TJSON, even though the project is not hostile to editing, that's not the point - the point is for a computer to take JSON, pump it into TJSON, and get an unusually human readable format that can be consistently converted back into JSON.  The point of the bare string and bare key forms is to produce text that looks maximally readable to humans.  If it's not going to be readable due to its content, feel free to fall back to double quoted forms even when not required - if it's going to look like code anyway there's no reason to force the bare forms.  While it is a hard requirement not to lose data and to be round trippable to JSON that represents exactly equivalent data, it is likely that in many use cases it will be a read-mostly format, and that's ok.

We should usually have a pretty good idea of where we are in the data structure just by looking at where we are on the page.  In some other standards, including JSON, knowing what level you are at requires counting various delimiters that may be several screens away from the line that you are looking at.  Because this is inherently difficult, the viewer ends up counting indents anyway in order to discern their location.  As long as the viewer is likely to end up counting indents anyway, why not make them reliably mean something wherever possible?  In TJSON, if we are at 10 spaces indent, we are five levels deep.  This is something that we can rely upon without looking around the document in most cases.  Also, because the indent level is always 2, we can actually see small discrepancies in it reliably with our naked eye, which is not the case for larger or optional indent steps.

Rigidly spaced outputs can sometimes help not only the viewing experience, but the editing experience as well.  Some subtle mistakes are best avoided by making them unlikely to parse.  In TJSON, the very position on the screen of the start of the text carries meaning.  Tolerating small variations in those spaces is not helpful to anyone, as doing so both degrades the visual experience by depriving it of meaningful demarcations, and makes mistakes more likely to carry hidden and erroneous meaning.

The parser must be able to parse all valid TJSON to JSON that represents the exact same data that the original JSON represented without any options on the TJSON parser, no matter what readability preferences were used with the generator to create TJSON from the original JSON.  Valid numeric representations in the original JSON must be preserved when translating to TJSON, and vice versa.  Exponent 'e+' vs just 'e' need not be distinguished as the meaning is identical and the look is nearly so.  Duplicate keys are not preserved by most JSON parsers in the wild, and are beyond the round-trip data fidelity guarantees that we try to enforce.  It doesn't mean that implementations should not try to preserve duplicate keys, but they can be valid TJSON implementations even if they do not.  If the platform preserves duplicate keys, the TJSON implementation should too, but is not required to do so.

Though a more text like presentation is less forward about type information, it should be possible for the user to see the data as much as possible, and with a bit of knowledge, to be able to discern the type.  Spaces at the end of lines are invisible to the user.  Also, part of our round trip path might flow through text editors that tend not to be happy about spaces at the ends of lines.  As such, spaces at the end of lines in the output, though not forbidden, are never necessary for proper parsing, and are avoided wherever possible.

In TJSON, indent level and spacing reveal the underlying data structure and types, and any explicit markers must highlight the truth of the existing space rather than overriding or supplanting it.  If something is in the wrong place, it's not the right thing in the wrong place, it's just lying about the data structure, as the location is the highest priority fact.  When type information is made more explicit than usual through non-default generator options, it must emphasize and reinforce the existing visual layout and visual cues.

TJSON is supposed to be flexible for generators.  Many of the fields have more than one way to express them, with guidelines for generators to use to pick which is most likely to look good.  Where you need a single representation that passes a hashsum even from another generator, we have CANONICAL TJSON - which is not the default.  CANONICAL TJSON is TJSON with many of the default optional readability features turned off to avoid diff churn, and to create byte for byte consistent TJSON between generators.  It's still much more readable than JSON in my opinion and is still a huge visual upgrade if you have lots of JSON.  It also may look better or be more readable for certain particular use cases even if you are not sensitive to diff churn, depending on your data.

The main competition for TJSON isn't JSON.  It isn't YAML, it isn't XML, or TOML, or anything else.  It's text.  It's the millionth and first time someone wrote an ad hoc printf, println, etc. statement to format their data, and the millionth time someone else has attempted to come up with a way to reliably parse it.  Let's just do it in a consistent way.  You've got data, you need human readable output.  Other people have your program and want data.  Make it easy for everyone.  Allow the interchange format to be a comfortable read for a human, and even masquerade as text to the unaware.  Put that printf down.  You've got alternatives.

---

## CONSUMERS


1) JSON COMPATIBLE DATA THAT LOOKS MORE LIKE REGULAR TEXT - This includes both nontechnical people that want to understand what is included in data, and technical people that want to be able to read their logs or treelike data more comfortably, or just without scrolling forever.

2) JSON THAT IS MORE EFFICIENT WITH YOUR SCREEN SPACE AND TAKES UP LESS VERTICAL SPACE - TJSON is good when you just want to be able to read and understand JSON without it looking like garbage or taking excessive amounts of vertical space (logging for example), and when you want JSON to be readable as text without looking like JSON.  These seem incompatible, but after much reflection I actually think they are very similar problems.

3) HUMANS WHO WANT TO READ JSON WHILE MAKING THE DATA STRUCTURE LESS NOTICEABLE, BUT NOT INVISIBLE - This includes pretty much everyone that has to look at a lot of moderate complexity JSON that contains large portions of human readable data.

4) ANYONE WHO WISHES THEY COULD WRITE BOTH DATA AND TEXT AT THE SAME TIME - Usually people pick one, but want both.  They usually never get around to the one they didn't pick.  But you don't have to pick.  Your data is probably treelike, and could be easily expressed as JSON.  Maybe it even is JSON.  But no one wants to read JSON.  Yep.  It's hideous.  I understand.  But what if I told you you don't have to choose anymore.  You can output JSON, and feed it into a small library and most people will not even be able to tell the difference, but for those in the know, it's data.  You can tell people that can barely boot a computer to paste it in a field, and even ingest it straight out of the database afterward without anyone feeling like they did anything at all.  You can have both.

5) EVERYONE THAT'S EVER WRITTEN AD HOC PRINTF OUTPUT FOR THEIR DATA, OR WRITTEN A PARSER FOR SOMEONE ELSE'S AD HOC PRINTF OUTPUT - Since the stone age, programmers have made up ad hoc rules to decide how to format text console output, and other programmers have had to make wild guesses as to what the parameters of those rules were when trying to parse it in order to turn it back into data.  Often it isn't deterministically parsable at all.  If you use TJSON, you KNOW it's deterministically parsable, and you know that your users can trivially make it back into data.

6) STORING JSON IN A HUMAN DOCUMENT WITHOUT LOSING INFORMATION - Embedding JSON in a human document, while allowing it to be turned back into the data that represented it without loss.  TJSON is great for the situation where you need something that is readable as text but also needs to be extracted later on a data layer as standard JSON.

7) EMBEDDING LIMITED AMOUNTS OF JSON IN AN ENTIRELY HUMAN DOCUMENT WITHOUT MAKING IT LOOK LIKE CODE (OR AS LITTLE AS POSSIBLE IF YOUR DATA STRUCTURE IS COMPLEX) - Avoiding all code like appearance is possible, as long as you are careful about the depth of your data structures and what you have in the JSON.  If you make sure not to descend more than one array or object level at once, it will become entirely invisible in many cases.

I expect there are also going to be other use cases I can't foresee right now, as pursuit of maximum readability has resulted in a shared format that programmers and non-programmers can both love for slightly different reasons, and that's a good thing.

---

## ANTI-GOALS

1) Making it easier for humans to write if it interferes with reading.

2) Anything that will make data not round trip from JSON.  This excepts comments, which have no JSON representation at all, and details that do not change the data, such as exactly which characters were escaped rather than literal in the original JSON, or the non-data indenting or spacing within the original JSON.

3) Anything that will make our output look more like code garbage than it has to - the point is to be as close to the regular human reading experience as possible without taking away the ability to understand the underlying JSON structure.  Certain things are going to look like random garbage no matter what we do, those things will be left alone.  If you have a kilobyte of random punctuation in a string in your JSON, it's pointless to try to change that to make it more human readable.

4) Making changes in the specification of the format to make code run more efficiently, or to make it fit some theoretical computer science grammar category.  This is for humans first, and computers are fast, it's ok if the code has to work a little harder to make it readable.  We do try to keep the code as fast as possible given the human first constraint, and we have a Rust implementation, so the performance is there.

5) Specifying folding rules to stay within pathologically small fixed width values, or for where the indent level n is almost the same as width (an indent glyph is provided to avoid the indent level getting too close to width for deep objects).  At very small width values, readability has already been lost.  What is being displayed is not analogous to human readable text anymore, and this specification will not attempt to accommodate it without overflowing a fixed width.  This avoids the insanity of deciding where to fold a long unicode code point, or a boolean.

6) Encouraging or requiring every platform to have its own general purpose implementation written from scratch, each with their own bugs.  This would be a recipe for disaster.  It's important that TJSON be well-specified and have platform independent tests so that special purpose implementations can exist, and so that other general purpose implementations can be made, but at least early on it is important that the reference implementation is linkable from everywhere without using that as an excuse for under specification.  Buggy special purpose generators or parsers meant for specific uses only are much less of a threat because it's easy to simply locally convert TJSON to JSON and then back to TJSON with a non-buggy implementation.  The first implementation needs to be flat out awesome.  It is sufficiently difficult to implement a correct general purpose parser or general purpose generator that everyone everywhere needs to be able to access a quality implementation without much effort.  As much as possible, all languages need access to a general purpose implementation that is fast, reliable, and runnable from anywhere that can handle JSON with a shim layer at most, both for ease of integration, and in order to avoid encouraging the creation of buggy general-purpose implementations that might cause fragmentation or generate things that they cannot parse.  This also has the added benefit that if the specification needs to evolve, it can do so with relative ease.

7) Replacing JSON - I have no desire to replace JSON, that would make no sense at all, as it is a wonderful computer to computer data format.  As described below, MINIMAL JSON (basically minimized JSON) is actually also valid TJSON.

---

## ABSTRACT GOALS OUTSIDE THE SCOPE OF THIS SPEC

There are some interesting ideas contained here that may be portable to other contexts.  I hope that people will take inspiration from this document in order to aid both data transparency and round trip ability in other places.

When a format is chosen for human readability, having to use an escape character often means that you made bad data format choices up until that point.

Bare strings, and bare keys here are portable concepts that others might be able to use in other languages in order to create human readable, round trippable tree data.

Some of the indent level parsing concepts in this document may also be portable, and spreading those ideas I think is important, even if it's well beyond the scope of the TJSON format.

Also, ideas about having multiple data preserving representations that read differently and are chosen between intelligently by the generator are also interesting, though by no means unique to this document (JSON pretty printers are already pushing that as far as the JSON specification allows), but I do think it's an interesting and useful idea that the data format itself ought to provide choices meant for human transparency, which I think will become more and more important in our LLM driven world.

One of the problems with JSON pretty printers is that even though they are impressive, and the authors are ingenious, they tend to be very fragile and degrade when you take them out of whatever narrow range of data structures that they were designed for.  That is due to limitations in the JSON format itself, not any lack of ingenuity of the pretty printer authors.  You end up getting a few spectacular best cases that fall apart with more varied real data, due entirely to the limitations of the JSON format itself.  It is important to recognize the visual straitjacket for what it is.

I've chosen to exploit what I think is an underexplored part of the design space of text/data file formats in TJSON, exploring how much we can get out of human perception if we discard the limitation that it must be easy to edit.  Up to this point I think we've often treated data formats as if it's about HUMANS or MACHINES, and made a compromise between HUMANS and MACHINES, but it's not really, it's about HUMAN READABILITY, HUMAN EDITABILITY, and MACHINES.  I'm sacrificing human editability to see how far I can push human readability without losing any information to a machine whatsoever.

---

## FUNDAMENTAL RULES

### Start and End of TJSON

THE START AND END OF TJSON is well defined, but beyond the scope of this document - so the interpreter will know to start reading the TJSON at a certain fence and stop reading it at another fence.  Extra empty lines at the end shouldn't break the parser.

TOP LEVEL TJSON STARTS AT INDENT 0, or another indent level as defined by the parser (possibly read from a fence, but that's beyond the scope of this document) - so we can assume that when we call our TJSON reader/writer we will have a starting indent level (that will practically speaking always be 0, but might be some other fixed offset that the caller can remove first, or maybe we will eventually trivially add a first pass to remove indenting in our parser, but that isn't a big deal if we ever need to do it outside of the parser).

There is only one root level value in TJSON at indent 0.  More than one is not allowed.

### EOL Handling and Post-Processing Resistance

TJSON is intended to be a substitute for text in many places, and as such it needs to allow for post-processing resistant features so it can survive alongside text, or within toolchains designed for text.  This does not mean that all generators need to resist post-processing, but it is important that an appropriate generator can produce post-processing resistant output as one of the goals of the format is that TJSON can at least potentially be embedded in text.

INCOMING UNESCAPED WINDOWS TYPE CRLF AND UNESCAPED LF ARE BOTH TREATED AS EOL, even in the same file.  A bare CR (carriage return not followed by LF) is a parse error - our parser should never see a bare CR.  If it does, that's an error.  TAB characters are only allowed as literal content inside multiline strings. A tab anywhere else is a parse error.  Implementors may choose to avoid multiline strings that contain tabs and make them regular JSON strings instead.

WHEN CREATING TJSON, OUTGOING EOL MUST DEFAULT TO LF NOT CRLF, unless it's incompatible with your use case or the user specifically requests otherwise through an option, in which case CRLF is EOL instead.  No other EOL's are allowed.  (As of this writing, most Windows tooling deals with LF just fine, so incompatible means more than just on Windows.)

TJSON HAS NO ZERO-LENGTH LINES WITHIN IT, ASIDE FROM WITHIN TRANSPARENT TYPE MULTILINE STRINGS.  Transparent multiline strings can contain empty (zero-length) lines; the bold multiline can't ever be an empty line because of the initial `|`, and the minimal can't because the interior of the string is at n+2, so you are guaranteed at least two spaces at the start.  Transparent type multilines are optional, allowing users that need no completely empty lines ever to easily avoid them.  Bold multilines only will guarantee no lines with only spaces as well if that is desired by the user, possibly to assist post-processing by other tools that might use such a line as a delimiter.

COMMENT DELETION OR ALTERATION USING EXTERNAL TOOLS WORKS, ASIDE FROM WITHIN NON-BOLD MULTILINES.  Multiline strings can contain '^[ ]*//.*$' comment lines that are actual data.  Anywhere else, a line starting with any number of spaces (including zero) followed by '//' is guaranteed to be a comment that can be stripped without changing the represented data.  If you intend to strip '//' comments using simple regex based tools, make sure to only use the `` pipe guarded multiline format (bold).  If you know the comments that you want to strip have a '//' at the start of the line, you can safely use the ` light multiline format too, but not ``` transparent type multilines as those can have a '//' at the start of the line that represents data.

TJSON SURVIVES LF->CRLF AND CRLF->LF CONVERSION OF THE WHOLE FILE WITHOUT LOSING BYTE PERFECT ROUND TRIP DATA SAFETY.  We preserve the unescaped data EOL type in multiline strings with a human-readable \r\n or \n (the default if omitted) as part of the initial multiline glyph, so we can mutate the file EOL between LF and CRLF without losing the data EOL.

DATA EOL TYPE IS USER VISIBLE.  The human readability of the escape sequence is important so the user reading it knows what the data contains within the multiline.  The user can tell what the line ending type is of a multiline just by reading it, similar to how they would be able to with a double quoted string with \r\n or \n inside.  The user reading it shouldn't have to guess line endings within data, and the parser knows what the data should be even if the file gets converted between EOL formats.  The user will not necessarily be able to spot spaces at the end of lines in a multiline, but that would also be true of the original document.

PRE-EOL SPACES MAY NOT SURVIVE TRIMMING, BUT CAN WITH THE CORRECT OPTIONS.  We do not guarantee we always survive line end space trimming by default as it can lose space data within multilines or folds, but multilines and folds are both optional, leaving a way for the user to pick options that allow TJSON to consistently survive line-end space trimming in hostile environments without losing data.

TRAILING SPACES ARE TREATED AS ERRORS BY DEFAULT WHERE NOT MEANINGFUL.  The reason for this is that sometimes they are meaningful (within multilines and occasionally within double quoted folds), and it's much better to find out that something is adding spaces to the end of your lines immediately as soon as you parse it rather than months later when you find out all of your multilines have had their data silently changed.  A special purpose parser or a general purpose parser with an option could choose to ignore this rule though, perhaps to demangle data that has already been mangled, or because we know that a human is editing it manually so we can chalk it up to that.

### Generator Options

THERE ARE MANY PLACES WHERE THE FUNCTION THAT CREATES THE TJSON HAS OPTIONS - It is ok for it to pick something that looks good for the use case as long as it outputs valid TJSON - which one to pick is a suggestion - generally bare is preferred for readability but a particular parser that had a reason would be allowed to do something else, but shouldn't do it without a reason - you might want to make your own parser that made output that looked good with your dataset and that's absolutely acceptable, but don't willy-nilly do something that's not preferred unless you think you are getting something out of it

### Forbidden Characters

FORBIDDEN CHARACTERS ANYWHERE IN TJSON - these MUST be escaped, EVERY TIME IN EVERY CONTEXT, INCLUDING MINIMAL JSON (escape them) AND MULTILINE STRINGS (fall back to escaped JSON STRING representation).  They are not allowed in comments either.  Just because they are FORBIDDEN doesn't mean we shouldn't ingest them, but we should never output them without escaping them.

1) Control characters other than TAB, and recognized EOLs (LF and CRLF): forbid `[\u0000-\u0008\u000B-\u000C\u000E-\u001F\u007F-\u009F]`
2) Default-ignorable code points `[\p{Default_Ignorable_Code_Point}]`
3) Surrogate code points `[\p{Cs}]` except for surrogate pairs that are as follows (the JSON spec allows this and we will allow here too):
   a) Unpaired surrogate code points — isolated surrogates `[\p{Cs}]` are forbidden. A high surrogate (`\uD800`–`\uDBFF`) immediately followed by a low surrogate (`\uDC00`–`\uDFFF`) as a `\uXXXX\uXXXX` escape pair within a JSON STRING is allowed, as it encodes a valid non-BMP Unicode code point. Literal surrogate bytes in the text are always forbidden regardless of pairing. (THIS RULE IS STRAIGHT FROM THE JSON SPEC AND IT'S A GOOD RULE TO USE HERE TOO, we can keep some emojis.)
4) Bidi characters `[\p{Bidi_Control}]`
5) Private-use code points `[\p{Co}]`
6) Non-characters `[\p{Noncharacter_Code_Point}]`
7) Line separator and paragraph separator `[\u2028\u2029]`

A JSON STRING or MINIMAL JSON are forbidden from having data characters that are unescaped `\r` or `\n`.  This is not an additional restriction, it flows from the JSON spec itself, but I would forbid them if they were not already forbidden by the JSON spec itself in those two contexts.

### Indentation

n is an integer indent level.  It starts at 0, it is incremented by +2, and decremented by -2.  It is FORBIDDEN for n to be an odd number.

Logically, the rule is starting any value creates a +2, usually transiently.  This includes a `[`, `{`, true, false, null, starting a number, or starting a string of any kind including an object key bare or otherwise, creates +2 indent.  This is almost never visible unless the value is folded.  Starting MINIMAL JSON creates +2 indent too (the content is entirely ignored for indentation purposes).  MINIMAL JSON cannot be wrapped or folded, so this ends up being entirely theoretical, but it is still the rule.  Folding a basic type like `""` or `{}` or null is just silly and not allowed, but that's the logical rule.

If you know you will never have to produce an actual indent while it's incremented, it may be unnecessary (we will never produce an indent in the middle of a `""`, `{}`, true, false, null, or `[]`).  Sometimes we exhaust an array or object completely within one line, in which case we are +2 when we start displaying it, and back to -2 by the end of the line, so the user never has a chance to see it, as we never actually print an indent while it's incremented, but that's the rule every time.  Folding actually makes the mid string +2 visible as it can also create a visible indent.  This means that a string that is folded 3 times is +2 each line (because logically it was +2 when we started the string), and -2 at the end of the string returning to where we started.  Another way to think of this is also that starting a string creates +2 indent level, and ending one creates -2, but this practically speaking never matters unless we are folding, as we never have a chance to produce an indent.  Implementations do not have to track +2 indent for the `{` and -2 indent for `}` in an empty object - as that would be insane as we would never be in a position to actually produce an indent between `{` and `}`.  Implementations also do not have to track indent levels other places where we know we are not going to actually have to produce an indent.

---

## PRIMITIVE VALUES

### BASIC TYPES: (true,false,null,[],{},"")

Null (this is the ONLY way to express null):
```
null
```

Empty array (this is the ONLY way to express an empty array, but see parsing note below):
```
[]
```

Empty object (this is the ONLY way to express an empty object, but see parsing note below):
```
{}
```

Empty string (this is the ONLY way to express an empty string):
```
""
```

Booleans (this is the ONLY way to express JSON true or JSON false (as distinct from the strings "true" and "false")):
```
true
false
```

Parsing note:  There is a small caveat for [] and {}, as [] can be an empty table, and {} can be an empty table row, but we should never generate either of these by default, even though we will parse them to allow extremely specialized generators to function.  One could imagine a generator that only ever outputs tables with certain keys, for example.

TJSON is case sensitive.  For example, 'True', 'False', and 'NULL' are parse errors.

### Treatment of String Representation of Basic Types: ("true" "false" "null" '""' "[]" and "{}")

"true" "false" and "null" are allowed as BARE STRINGS by default, and in principle are allowed as BARE KEYS (but bare key rules forbid "", "[]", and "{}" already, independent of their status as BASIC TYPES, so these are not possible, and BARE STRINGS already forbid "" as it both starts and ends with a quotelike, "[]" as it starts or ends with a SQUAREBRACKETLIKE, and "{}" as it starts or ends with a CURLYBRACKETLIKE).

Implementors may choose as an option or consistent with their particular use case to force double quotes around some strings.  Strings that are the exact string representation of a BASIC TYPE ("true" "false" "null") are and should be allowed BARE STRINGS by default, but an implementor might force these particular strings only to be double quoted, or even forbid bare strings entirely (strongly discouraged, definitely not a good idea by default, BARE STRINGS are preferred).  An implementor might forbid the string version of some or all of the BASIC TYPES above ("null", "true", "false") from being bare strings in tables (not the default).  Such strings must be parsable anywhere as bare strings, including in tables.  The spec intentionally does not force the strings null, true, false, {}, and [] to be double quoted JSON type strings rather than bare strings unless the user explicitly chooses that option.

If your generator actually uses MINIMAL JSON as more than as a last ditch effort, that generator should probably have different rules for when to use a bare key than the default generator.  If a generator actually uses MINIMAL JSON more than in extremis, for stuff like `  barekey:["minimal","json"]`, the generator should limit the usage of bare strings for visual analogs, by forcing double quotes on anything that starts or ends with square brackets or curly braces, and any close unicode analogs that might mislead a reader.

Some users are going to have a higher need to avoid adversarial examples than others, and may be willing to tolerate more visual ugliness in order to achieve that goal.  They should make different generator choices than those who don't have that issue.  Good choices for this are to force `` ` `` type multiline strings if you use multiline strings at all, and to have much more conservative rules on when you generate a bare key or a bare string.  Such users may also want to have their own parser options to refuse to parse anything that doesn't match their narrower rules.  Such a parser is not going to be within this specification with those options, and cannot be considered a valid TJSON parser, but it might be a good idea to use internally in certain circumstances.  If you have an exceptional need to avoid adversarial examples, you might even forbid all bare keys and bare strings, forcing double quoted versions instead, and refuse to parse anything else, but at that point you might also consider an entirely different format instead.

### Numbers

Numbers (this is the ONLY way to express numbers):
Number any valid JSON number (0, 0.1, -5, 1e-3, -5e10 are all good), note that the characters allowed here are very constrained by "any valid JSON number", particularly at the ends where it would otherwise break things.

### Comments

Comments (this is the ONLY way to express comments):
Comments are allowed, they must start with a `//` and be on their own line.  A comment cannot be within a MULTILINE STRING to include its starting and ending glyphs.  A comment may not be within a fold.  Comments are allowed to have any number, or 0 spaces before the `//`.  Anything after the `//` is ignored until we hit an EOL character.  Comments do not need to respect the indent level, and they do not affect the indent level irrespective of how many spaces they do or do not have before.  Comments at the end of lines containing anything else are not allowed.  There is no other form of comment.  A comment of course can never round trip with JSON, and will probably rarely get used as this is a read almost always format, written almost exclusively by computers, but they are allowed.  It made sense to exclude them from the first character of bare strings and bare keys anyway as they looked like comments, so we might as well make them actual comments if we have to reserve `/` anyway.

#### Adjacent Comment Indentation Rule

There is a special rule to preserve comment adjacency and line order after rerendering the same data, and that is that for any two adjacent comments, the second one cannot have a lower indent than the comment immediately before it.

TJSON is inherently spatial by design, and it makes sense that people who want to comment a particular thing where there is more than one thing on a line would put it adjacent spatially at a visibly similar indent level.  Parsers that preserve comments may make the same coupling also.  This is something we want to preserve, but we never want it to conflict with document line order.

If the comment indent went down from one line to the next, it could indicate a correspondence with data that was in the opposite direction of the lines in the file.  If the comments were associated with data in the file by a parser, and the data on the line moved during a rerender, it's possible that for the comment to stay with the data it appeared to be associated with, the line order in the file would have to invert.  In order to stop any parser from ever having to do this, we simply parse error whenever consecutive comment lines have a lower indent than the comment immediately before.  This does not necessarily mean that comment lines need to have any particular relation with other comment lines as to indent level, or any relation with the data indent level.  They could.  A parser might reasonably assume that.  But it isn't required.  It also imposes no constraint whatsoever on comment lines that are not immediately adjacent.  It imposes no restriction on the range of comment indent allowed levels, a comment may have any indent level at all irrespective of what data it is near, or what other data is in the same file.  This is a very narrow restriction that applies only to adjacent lines.  This restriction has nothing to do with anything that goes after the `//`.

Here's an example of why this rule matters to a spatial format like TJSON.

```json
[[[["bare string"]]]]
```

*Bad Example* (parse error)
```tjson
// this is a comment about the outer array (1)
  // this is a comment about the next array level (2)
// this is another comment about the outer array (3)
      // this is a comment about the innermost array level (4)
[ [ [ [  bare string
```

*Good Example*
```tjson
// this is a comment about the outer array (1)
// this is another comment about the outer array (2)
  // this is a comment about the next array level (3)
      // this is a comment about the innermost array level (4)
[ [ [ [  bare string
```
*Valid Example* (For if you stylistically want the declining indent but don't want a parse error)
```tjson
//       The parser
//                        doesn't care
//   what you type after the '//'
// at all.
//
// The parser doesn't care what comes after the //, it only looks at the indent before it.
                                                                 // this comment is fine, it's at the same or more indented, even though it's well past the data.
[ [ [ [  bare string
                                         // this comment is fine too, it's not adjacent to the other comments, so it can be anywhere.
         another bare string inserted after bare string
```

If we didn't parse error on the first one, a smart document parser that preserves comments would have to do this when you inserted data, which is a reordering that might be surprising to the reader.  If it didn't do that, it would lose the data pairing just because information was added.  Not every parser has to data pair comment lines spatially like this, but a parser shouldn't have to choose between spatial data association and losing line order of adjacent comments.  In order to avoid this potential, we simply reject adjacent comments that lose indent when we parse.

*Example: What would happen if we inserted stuff in the bad example above and didn't have this rule*
```tjson
// this is a comment about the outer array (1)
// this is another comment about the outer array (3) NOW OUT OF ORDER!
[  inserted in outer array
  // this is a comment about the next array level (2)
  [  inserted in next array level
      // this is a comment about the innermost array level (4)
    [ [  bare string
```

A parser isn't required to be this smart.  It would still be a parser (though perhaps not as good) if it simply associated the commented line with the first thing on the next line and treated adjacent comment lines as if they were all at the same indent even when the indent is increasing.  It would probably be worse, but it would still be a valid TJSON parser.  But even if it's dumber, it shouldn't have to decide between data correspondence and reordering lines.

It's also important that this rule be easy to apply for a relatively dumb parser, which is why we apply it even if we wouldn't otherwise reorder the lines.  Maybe two comments are so close that even if you were associating them with data, it would be the same data, maybe it's only one column.  That shouldn't matter.  It's always simple: if the number of spaces in the indent decreases over two consecutive comment lines, it's a parse error.


---

## OBJECT KEYS AND STRINGS

OBJECT KEYS: (this is not part of STRINGS in this document)
OBJECT KEYS have two choices that the generator can pick from, the default is to prefer BARE KEYS everywhere it's allowed.

### Bare Keys

BARE KEYS (preferred by default, even when it's also a basic type like "true" "false" or "null" (the bare key rules already forbid "{}" and "[]") - this is even safer than it is for bare strings as keys in JSON are always strings reducing the likelihood of confusion)

BARE KEYS are always the same or stricter than BARE STRINGS.  Any restriction on a BARE STRING also applies to a BARE KEY, but bare keys have additional restrictions.  There is no valid BARE KEY that cannot be a valid BARE STRING in the same context.  If the spec allows this (and it shouldn't) it's a bug in the spec.

BARE KEYS are required to meet certain rules, which are expressed below - not everything is allowed to be a BARE KEY

Valid BARE KEYS MUST satisfy all of the following rules: (MUST - this is not optional):

0. Must satisfy all the rules for BARE STRINGS.  In a table heading, a bare key must satisfy the rules for bare strings in a table also.  All other rules are in addition to this, not in place of this.
1. `/^[\p{L}\p{N}][\p{L}\p{N}_()/'.!%&, ;@$#*=?^~<>+-]*$/u`
2. Start with a letter or number (already implied by regexp, stricter than bare strings)
3. Other chars after the first must be `[\p{L}\p{N}_()/'.!%&, ;@$#*=?^~<>+-]` (already implied by regexp)
4. The first or last character cannot be a space, a PIPELIKE CHARACTER, a COMMALIKE CHARACTER, or a QUOTELIKE CHARACTER (look at rules for bare strings, last char here must follow last char rules for bare strings too (we are already way tighter than that with the regexp but it does remove `,` and `'`) (not implied by regexp, though the regexp does exclude some but not all of these from the first character already)
5. May not contain more than one consecutive space (not implied by regexp, but implied by bare string rules)
6. May not contain a COLONLIKE CHARACTER anywhere (not implied by regexp, some are in p{L}, this is narrower than bare strings which are allowed to have a colon.)
7. May not contain any weird characters anywhere other than regular interior non-consecutive spaces (fully implied by bare string definition): `[\p{C}\p{Z}\p{M}\p{Default_Ignorable_Code_Point}]`

### JSON String Keys

A JSON double quoted object key (a JSON STRING) that is allowed to contain escape characters and is exactly equivalent to a double quoted JSON object key.

In this document, object keys are just keys, and the default favored form of them is BARE KEYS wherever possible (described later), falling back to JSON type double quoted keys.  This is described more later in the document, but is added here just to clarify that the STRINGS rules below should never be applied to object keys.  An object key can be folded.

### Strings Overview

STRINGS: (not to include object keys, addressed later)
Strings can have a variety of formats and this is intentional.  The default favored format is the BARE STRING.  In Json, a key is always a string, so there is no need to disambiguate it with the bare string type notation.  A key is either a JSON STRING or a BARE KEY, it's never one of the string types below.


### Bare Strings

A BARE STRING cannot start or end with space, and cannot have whitespace at all other than non-consecutive regular spaces internally.  No space space allowed, multiple spaces are fine as long as they aren't at the ends and aren't next to each other.

A BARE STRING also cannot start or end with QUOTELIKE CHARACTERS of any kind, to include backtick (U+0060), see definition of QUOTELIKE CHARACTERS), and cannot start or end with COMMALIKE CHARACTERS (`[,\uFF0C\uFE50...]` — this includes ASCII comma U+002C, fullwidth comma U+FF0C, small comma U+FE50, and others, see COMMALIKE CHARACTERS).  This is to avoid the user thinking something is quoted or in an inline array when it isn't.
A BARE STRING also cannot start with a `/` or FORESLASHLIKE CHARACTER, to avoid looking like a comment or a fold marker when it is not. A BARE STRING cannot start with a underscore or UNDERSCORELIKE CHARACTER to avoid looking like a `_`, which can be used at the beginning of a bare string in lieu of a space when you want to be extra literal, but the generator will not produce it by default.  A BARE STRING also cannot start or end with a `|` pipe or PIPELIKE CHARACTER, to avoid it looking like the edge of a table line or a MULTILINE STRING when it is not.  For similar reasons, a bare string is not allowed to start or end with a `[` `]`, `{` `}` or analogs SQUAREBRACKETLIKE CHARACTERS and CURLYBRACKETLIKE CHARACTERS, and the limitation is not sided, in that neither side of a square bracket or curly brace can appear at the beginning, and neither side of a square bracket or curly brace can appear at the end.  The point of this is to avoid confusing the reader, who might otherwise think it is an indent marker, coupled with an indent marker, or part of MINIMAL JSON when it is not.  This also happens to exclude the empty array and object, though that is not the primary reason for the exclusion.

Implementations may want more exclusions than these, they are not required to represent anything as a bare string, but they do need to be able to parse it.  The whole reason we have bare strings is to make them readable.  Except for our regular space (U+0020) rules above, control characters, invisible characters, composed characters (the problem there is not that they are composed, but that when you see a character in a bare string you can't even theoretically know its unicode representation anymore by looking at it, by restricting it to the single character only representation, now we can at least theoretically know) and other weird characters are forbidden (`[\p{C}\p{Z}\p{M}\p{Default_Ignorable_Code_Point}]`).  We intentionally allow emojis with exactly one bare string representation, this is not a mistake.  It's ok for programs that create TJSON to be more strict than this even by default (it's pretty generous, deterministic emojis for example) and fall back to a JSON string with some of the characters allowed by this rule, but they can't be less strict.  Unfortunately, excluding `\p{M}` does exclude certain languages, but without the exclusion the risk of hiding data is too great.

#### Enumerated list of certain disallowed ASCII characters at the ends described above (not including space)
This list does not expand or contract the rules, it's just a convenient enumeration of some of them.  It's not exhaustive, but it's an easy way to keep the starting and ending rules straight.

BARE STRING START ONLY DISALLOWED CHARACTERS:
ASCII:
`_`
`/`
NON-ASCII UNICODE: analogs of above, see UNDERSCORELIKE/FORESLASHLIKE

BARE STRING START AND END DISALLOWED CHARACTERS:
These are all meaningful structural characters:
ASCII:
`,`
`` ` ``
`"`
`'`
`|`
`[` (both sides, in any direction, corresponding or not)
`]` (both sides, in any direction, corresponding or not)
`{` (both sides, in any direction, corresponding or not)
`}` (both sides, in any direction, corresponding or not)
NON-ASCII UNICODE: analogs of above, see PIPELIKE/QUOTELIKE/COMMALIKE/SQUAREBRACKETLIKE/CURLYBRACKETLIKE

If a particular user spoke Tamil or Arabic and was willing to accept the risk internally, they might write their own generator option that allows some of this anyway, and may want to have their own nonconforming local parser option that is looser than the spec, but don't do this with any TJSON you aren't planning to consume exclusively locally, as it is not compliant.  However, the limitations on [[]{}"/:,|`] in bare strings and bare keys, and any of the regular space rules, should be a parse error even with some special case local only nonconforming parse/ingest toolchain as they are parse-critical.

Parsers are allowed to have a strict mode that rejects any unescaped character that is not allowed to be produced by the spec, and for a parser that only works in a certain context, a parser might choose to reject those characters entirely.  (A high security context for instance.)  If a parser chooses to do this by default it should inform the user if practicable that it is parsing in strict mode or similar by default.  A parser might also choose to add security features and refuse to ingest certain things entirely escaped or otherwise, but that is not specification compliant, and should probably be done at the data layer rather than in the parser itself, at least logically, as it's doing two things, being a TJSON parser, and a security layer, not just being a TJSON parser.

A BARE STRING (but not a BARE KEY) always has a non-data leading space (default), or optionally, in place of the non-data leading space, a non-data underscore when the generator is specifically asked to do so.  This leading space (default) or underscore (ugly, but allowed because sometimes an additional degree of emphasis on type is essential) is treated identically by the parser.  Because both a space and an underscore take up one space, and the parser doesn't depend on which is found, there is no difference in position between the two, ever.  They are completely interchangable, and no consistency in usage of one over the other in a document or even a line is required.  By default, generators should use the leading space, not the underscore.

#### Additional Bare String/Bare Key Rule for Tables Only

A generator is FORBIDDEN from producing a bare string or bare key that contains a `|` character within a table.  Generators should also avoid pipelike characters within a bare string in a table but this is a recommendation not a hard rule as it is not parse critical.  We want to be able to deterministically parse tables with insufficient padding, and having a hard rule on this allows us to do that.  It also allows special purpose generators to break padding rules without a parse failure if it makes sense for their use case, and provides assistance to people editing by hand.  We sacrifice bare strings in tables containing `|` to padding flexibility because bare string `|` is likely to be confusing, and insufficient padding is unlikely to be confusing.

Within a table, a parser MUST reject a bare string or bare key that contains a literal `|`.

String: (this is a BARE STRING, the main and preferred string type) space almost-anything-see-above space space (space space is the ending only if something else is packed after it, we don't put meaningless spaces at the end of lines, we use an EOL instead)

```json
".*"
```
```
 .*
```

String (end of line): space anything newline
```json
"string"
```
```
 string
```


String (alternate bare format, not the default bare format, with underscore in place of leading space to emphasize type, note that the characters are in the same place): space anything newline
```json
"string"
```
```
_string
```

### Folding

Folding isn't required, but it can be necessary where width is at a premium on a device.  In order to avoid pathological cases that are not meaningfully textlike, we will not even attempt to break things into fold chunks that are fewer than 10 visible characters wide in order to preserve a fixed width, or attempt to honor a width of less than 20.  We do sometimes make fold chunks smaller than 10 characters, but it's to make it look good, not to match a fixed width.  In many cases, our folding rules will not have a valid way to fold for lengths that are that short.  An example would be where do you fold false?  How about "\u10FFFF"?  It's possible to come up with rules for these things, but at that point we aren't really in the target area of this specification.  If you are doing that, it's not text anymore.

If the user picks a reasonable width, but the object is very deep, the way to handle that is to either overflow the width or use indent glyphs.  The solution is not such madness as attempting to fold false.

Fold indicators must be both preceded and followed by at least one data character.  Otherwise, infinite intermediate fold markers would be allowed in the middle of any string.  This does not prohibit empty rows in tables, as a pipe in a table context counts as a data character for the purposes of this rule.  A colon immediately after an object key counts as a data character for the purposes of this rule.  Neither an object key colon nor a table pipe counts as a data character generally.

### Number Folding

Number folding follows the same rules as string folding below, including the requirement that the fold must be between the first and last data character.  Numbers are unlikely to need folding as most of them are not long, but it is possible so it is specified here.  Implementations should usually do something else before they get close enough to the margin to have to fold a number, such as use an indent glyph (described later) or just overflow the set width, but folding a number is allowed.  Generators should try to avoid folding numbers, especially right after the initial sign as it's more likely to be confusing to the user.  Breaks in numbers usually look better right before the `e` if there is one, or before the `.` (not quite as nice as before the e), but this is a generator preference.

### Object Key Folding

Object key folding follows the same rules as string folding below (a bare object key being treated like a bare key, and a double quoted object key being treated like a double quoted string).  There is however, one additional caveat.

In the special case of a key value pair where the value is a basic type (true, false, null, string, number, empty array [], or empty object {}), the key and the value are folded as a single unit, with the fold marker allowed in two additional spots at the junction between the key and the following basic value: immediately after the intermediate ':' (preferred by default), or immediately before the intermediate ':' (ugly, avoid, but allowed).  The opening glyph of a multiline string can be treated as a basic value in this context.  Part of a folded object key is FORBIDDEN from being on the same line as any other value.

In order to create the visual impression that both the object key and the basic value that follow are a single unit we have a special indent rule for the intermediate state between the key and the value.  Even though we are as a special rule allowing folding adjacent to the ':', which might otherwise be considered at a lower indent level than the interior of the key or the interior of the following value, we treat folds adjacent to the ':' as occurring at the same n level as the interior of the object key, or the interior of the basic value in order to preserve visual consistency.

Another way of saying the same thing is that at the start of the object key increment n+2, and at the end we decrement n-2, and at the start of the basic value we increment n+2, and at the end of the basic value we decrement n-2, but logically the n-2 from the end of the object key and the n+2 from the beginning of the basic value are merged, and thus the intermediate n value can never show up in a fold or an indent.  This is important for both readability and avoiding parse ambiguity.


**Example: Separates key and value, but uses an additional line**
```
  justthewronglengthkey:
  /  This is a string directly on the object key
```

**Example: Separates key and value, but uses an additional line**
```
  justthewronglength
  / key:
  /  This is a string
  /  directly on the
  /  object key
```

**Example: Keeps key and value together on a line, may look worse, but may require fewer lines which is why a generator would choose it**
```
  justthewronglength
  / key: This is a
  /  string directly
  /  on the object key
```

**Example: Keeps key and value together on a line, may look worse, but may require fewer lines which is why a generator would choose it**
```
  justthewronglengthkey: This is
  /  a string directly on the
  /  object key
```

**Forbidden Example: (should not parse) (n is allowed to return to n-2 adjacent to the ':' before finishing the key basic value pair)**
```
  justthewronglengthkey:
   This is a string
```

**Forbidden Example: (should not parse) (n is allowed to return to n-2 adjacent to the ':' before finishing the key basic value pair)**
```
  justthewronglengthkey
/ : This is a string
```

### String Folding

All string folding, JSON string, bare string or bare key MUST be between the first and last data character.  In both the JSON string `"abcd"` and the identical bare string, ` abcd` the first and last data character are the "a" and the "d" respectively.  In the JSON string `"\ \ \ "` the first data character is the first space, and the last data character is the last space including its escape backslash immediately before.

FOLDING IN THE MIDDLE OF A DATA CHARACTER OR IN THE MIDDLE OF A VISIBLE CHARACTER IS ALWAYS FORBIDDEN IN EVERY CONTEXT (in this case `"\ "` is a single data character, and anything displayed together is also a single visible character, like an emoji for instance).  We could (but probably should avoid) folding in the middle of the escape sequence for an emoji smiley face 😀 `\uD83D\uDE00` by folding before the escaped `\uDE00`, but we couldn't fold in the middle of those two literal characters if they weren't escaped as the user would no longer see a smiley face when reading the TJSON.

THEORETICALLY, FOLDING IS NOT ALLOWED WHEN visible n = 0.  This is because there is no room for the fold marker.  However, in practice this is basically impossible, because within any value (though this is theoretical in any case except strings and numbers because we don't allow folding otherwise and thus may not bother keeping track of n within the middle of printing, null, or true or other basic values), n = n+2 until we hit the end, so even a root string or number can fold, it just gets indented when it does so.  The reason it's ok in a root string or number is that n=0 at the start, but when we start reading the value, n=n+2, if we fold now we are at n>=2 so we are fine.  We don't even need programming logic to check this because it's an impossible condition, but I wanted to put it here to make sure the theory is clean.

EXTRA BARE STRING FOLDING RULE: a bare string may never be folded immediately after a single consecutive space.  This is both to avoid whitespace at the end of the line (desirable but not a hard requirement) and to create a consistent look.  This is a hard rule, folding after an internal space in a bare string is FORBIDDEN.

FOLDING a JSON string in general should be BEFORE unescaped space runs rather than after, in order to avoid unescaped whitespace at the ends of lines where possible, to avoid hiding data, and to create a consistent look.  This is not a hard requirement for JSON strings.

**Example: bare string fold (word with no internal spaces)**
```
 thisisareallyreallyreallyreallyl
/ ongwordhere bare string sentence
/  with words
```

**Example: bare string fold (words with spaces)**
```
 this is a really really really
/  long text here bare string
/  sentence with words
```

**Example: bare key fold (words with spaces)**
```
  this is a really really long
  / bare key: start bare string
  / same bare string
```

**Example: JSON string fold**
```
"this is a really really really
/  long text here json string
/  sentence with words"
```

**Example: JSON string fold with leading spaces on continuation**
```
"this is a really really really
/  long text here with 5 spaces
/      sentence with words"
```

### JSON Strings

String (alternate): JSON STRING (note no space before)
LOGIC:  this can look good sometimes, but it's also used as the fallback when we just can't make it pretty, if we can't make it pretty, implementors should fall back to this.
```
"usesJsonString\n\rRulesIncludingEscapes"
```

String (alternate): JSON STRING (with folding)
LOGIC: we are folding, so indent n+2, JSON strings cannot contain a real LF, so we know we did it when we add a real LF, add n-2 spaces to the next line, then a fold marker with a space `/ ` then continue with JSON, n-2 once we get to the last `"`.  A fold SHOULD never be preceded by whitespace as it may get stripped but the parser should handle it.
```
"foldingat
/ onlyafew\r\n
/ characters
/ hereusing
/ somejson
/ escapes\\\ "
```

### Multiline Strings

We have three versions of multiline string starter glyphs, for different situations.  Note that all forms start with a space before the first backtick, just like a bare string.  A bare string cannot start with a backtick rendering this unambiguous to the parser.

We will give each of these three versions useful shorthand names.

The triple backtick form, we call TRANSPARENT multiline strings, the unguarded ` ``` `.  We call it transparent because if you started looking at it mid way through, you would have no way to tell that you were not looking at the original document, as it has no indent, and no guard on the intermediate lines between the starting and ending glyphs.  This makes it very easy to look at, but occasionally confusing depending on the content of the document.  A document that contained TJSON itself, while it would still parse, would likely mislead the reader.  Also, a document that happened to contain an ending glyph of this type would fail to parse correctly, and as such generators must check that the document itself does not have such a line in it before choosing this form.

The double backtick form, we call BOLD multiline strings, the pipe guarded ` `` `.  The body is guarded on each intermediate line with a `| ` within the last two characters of the indent.  The indent can either go at its natural indent level n, or it can go at any even number between n and 2, inclusive.  We can't go below n=2 within the multiline to make room for `| `.  n=2 within the multiline is exactly how it would display if the string were the root node of the document.  We call it bold because the leading pipe makes it extra prominent to the eye.  This also makes it a safe way to encapsulate otherwise confusing text.  The encapsulation also makes it useful in situations where you might need to encapsulate something that looks vaguely like TJSON, or when you might not want an interior empty line under any circumstances.  Bold is ideal for logging scenarios, and is generally the safest choice of the three.

The single backtick form, we call MINIMAL multiline strings, the unguarded normal indent `` ` ``.  This form cannot change the visible indent, unlike the other two types.  We call it minimal both because it is minimally obtrusive to the eye, and because it does the least as far as its ability to break normal parsing rules.  It's particularly good for short multilines, though the choice is left up to the generator.  Because of the guaranteed lesser indent of the ending glyph, this form can contain anything without parsing ambiguity.

All three multiline string forms can have an optional `\r\n` or `\n` or nothing as part of the starting glyph at the end of the glyph.  Whatever form of the starting glyph, the ending glyph must match it exactly and be at the starting unmanipulated logical n indent.  For all three ` ``` ` TRANSPARENT, ` `` ` BOLD and `` ` `` MINIMAL forms, the ending glyph is REQUIRED, and it is REQUIRED to be at the proper indent.  Any other indent, higher or lower, is a parse error.  This rule is especially important from a reading perspective to preserve the ending glyph if you change the indent; we need to draw the reader's eye back to the correct indent level at the end so that they can continue reading without becoming disoriented after the multiline ends.

Our starting glyph (the glyph includes the space at the start) is: `` ' `{1,3}(\\n|\\r\\n)?' `` EOL (+2 space indent until just before the ending glyph line) (though ` `` ` can and ` ``` ` must manipulate the indent level between the start and end glyph as defined below), the actual string data being displayed SHOULD contain at least one linefeed and is REQUIRED to contain at least one data character, and the final newline after we are done showing our MULTILINE STRING is at the original indent before we started and is not part of the string, ends when we see a final EOL followed by n spaces and our starting glyph including its leading space.  This is unparseable for ` ``` ` if the document contains that sequence, so another form of string (` `` ` is pretty good here, but you can fall back to a non MULTILINE STRING too) must be used instead.

The `(\n|\r\n)?` after the one to three backticks is a LOCAL EOL INDICATOR indicating what data any EOL (either `\n` or `\r\n`) within the MULTILINE STRING represents.  The default is `\n` if just ` ``` ` ` `` ` or `` ` `` is used.  Generators should default to displaying ` ``` `, ` `` `, and `` ` `` over ` ```\n `, ` ``\n `, and `` `\n ``.

Multiline strings are 100% literal after indent stripping between the starting glyph line and the ending glyph line.  Both the start and end glyphs which must match each other exactly can end with either nothing (same as `\n`), `\r\n`, or `\n` which is interpreted as our LOCAL EOL INDICATOR when reading it in to data.  This means that a multiline string when we encode it must contain only literal `\n` or only literal `\r\n` within it as an EOL character - otherwise it cannot be expressed in the multiline format and reading it will cause data loss, so we forbid the data from having both.  Strings containing both `\n` and `\r\n` must use the JSON string form instead.

Comments are not allowed within multiline strings.  This includes anywhere between the starting glyph and the ending glyph, not just the string itself.

When parsing multiline strings, both LF and CRLF must parse as the LOCAL EOL INDICATOR, even if they are both present.  Multiline strings in the ` `` ` or `` ` `` form (but not ` ``` `) can be folded by the implementor like JSON strings.  Generators should default to multiline strings not being foldable.  If we fold them and add an EOL, we must use a fold marker in the last two spaces of the indent `/ ` (`/ ` replaces the last two spaces of the indent of `| ` for ` `` `), but if it's native to the string, we leave the fold marker out (as it was not folded, it's just the data).  A ` `` ` multiline string MUST have a marker within the margin of `| ` (or `/ ` iff that line is folded).  A ` ``` ` or `` ` `` multiline string may not have a pipe margin marker.

MULTILINE STRING text characters are meant to be extremely broad, but the user must be able to see whatever text is contained in it.  It's ok if what the user sees isn't necessarily the exclusive way of displaying what the user sees.  If an accented letter might be expressed two different ways in unicode, that's fine in this context.  Similarly, if an emoji might display slightly differently or in an unexpected or identical way, that's ok too as the user can at least see an emoji.  The consequence of forbidding a character from here is forbidding an entire document, which is not desirable unless we have to do it for safety reasons.  Also, if a character affects layout like a newline, we aren't interpreting it as such, creating an inconsistency in what the user sees, so we ban line separator and paragraph separator too.

Implementors should probably choose to avoid MULTILINE STRING format when the document itself has stuff that looks like an ending glyph, but are not required to unless the document itself has a line that matches the last line including the ending glyph of a MULTILINE STRING exactly, in which case in some cases there's no way to cleanly parse it, and in the others it's tricky, so we just avoid it by mandating the ending glyph.  If we have to use another format, possibly even another MULTILINE FORMAT; ` `` ` is particularly clear when the document contains something that looks like a glyph, but implementors may decide that it is inadvisable for appearance reasons.

MULTILINE STRINGS do not allow FORBIDDEN CHARACTERS - we have no way to escape them in a MULTILINE STRING, so we just don't allow them at all, fall back to JSON STRINGS and escape the FORBIDDEN CHARACTERS in the JSON STRING.

A document consisting only of a root node of a single MULTILINE STRING cannot use the `` ` `` format of multiline string.  The reason for this is if we ever decide to add 4/6/8 indent levels instead of the default 2, we wouldn't be able to discover the indent level from the data, making it impossible for the parser to be certain what was and was not part of the multiline string.  This isn't the case for a root node array or object.

The closing glyph must always exactly match the opening glyph (including its leading space), with exactly n spaces of indent before it, whether or not the opening glyph was pushed over by being packed on the same line with something else.  The closing glyph is always mandatory.

**Sample: ` ``` ` TRANSPARENT form (unguarded, full-width), inline with key, LF document inside CRLF string**

JSON (it's long so most of it omitted in favor of ...)
```json
{"thisisabarekey":"This....capital A"}
```
```
  thisisabarekey: ```\r\n
This is a TRANSPARENT MULTILINE STRING the T in this is the first character in the string
The opener here is pushed to the right from the normal indent level by the bare key.  The implementor could have chosen to move it to the next line like the next example, but did not.
Note that the closer is at the logical n indent level we started, not at the level the opening glyph got pushed to by the bare key.
Folding is totally prohibited here no matter what.  There is no allowed way to fold in this type of multiline string no matter how long the lines get.
We start on the very first character of the line no matter how high n was when we started
We end with a newline, indent to old n indent depth and repeat the starting glyph to indicate the end.
This would not be allowed if the document itself contained five spaces ```\r\n at the very beginning or after any EOL as the user would not be able to see the end.
The last character in this MULTILINE STRING is a capital A
   ```\r\n
```

**Sample: ` ``` ` TRANSPARENT form (unguarded, full-width), value on next line, LF string**

JSON (it's long so most of it omitted in favor of ...)
```json
{"thisisabarekey":"This....capital A"}
```
```
  thisisabarekey:
   ```\n
This is a TRANSPARENT MULTILINE STRING the T in this is the first character in the string
Folding is totally prohibited here no matter what.  There is no allowed way to fold in this type of multiline string no matter how long the lines get.
We start on the very first character of the line no matter how high n was when we started
We end with a newline, indent to old n indent depth and repeat the starting glyph to indicate the end.
This would not be allowed if the document itself contained five spaces ```\n at the very beginning or after any EOL as the user would not be able to see the end.
The last character in this MULTILINE STRING is a capital A
   ```\n
```

**Sample: ` ``` ` TRANSPARENT form (unguarded, full-width), as first element of array**

JSON (it's long so most of it omitted in favor of ...)
```json
["This....capital A", 3]
```
````
   ```
This is a TRANSPARENT MULTILINE STRING the T in this is the first character in the string
Folding is totally prohibited here no matter what.  There is no allowed way to fold in this type of multiline string no matter how long the lines get.
We start on the very first character of the line no matter how high n was when we started
We end with a newline, indent to old n indent depth and repeat the starting glyph to indicate the end.
This would not be allowed if the document itself contained five spaces ``` at the very beginning or after any EOL as the user would not be able to see the end.
The last character in this MULTILINE STRING is a capital A
   ```
  3
````

**Sample: ` `` ` BOLD form (pipe-guarded), same indent as surrounding array**

JSON (it's long so most of it omitted in favor of ...)
```json
["This....capital A", 3]
```
```
   ``
  | This is a BOLD MULTILINE STRING the T in this is the first character in the string.
  | Notice we only use two backticks for this type of multiline string with a starting pipe.
  | Because it's guarded by a starting pipe, we can subtract 0 or any even number of indent n in the middle, or just leave it at the same indent.
  | This one is left at the same indent.  Backing up the indent an odd number would be FORBIDDEN.
  | It's ok if the multiline string itself contains backticks like this `` or ``\r\n or ``\n, because it's required to be guarded by a pipe so we can tell where the start is when parsing.
  | Folding is ok, here's how we do it. This is the allowed way to fold in this type of multiline
  / string no matter how long the lines get.  It's totally ok to leave spaces at the end of a line in any MULTILINE
  / string not just this one and it's often desirable when folding.  The fold character is in the margin, one space back from the last character of the margin
  / just like we do folding everywhere else including tables.
  | We didn't start this with a ``\n or ``\r\n, but we could have and it would be allowed.  Our ending glyph should match our starting glyph, here only a ``, but if
  / it were a ``\r\n, our ending glyph would have to be a ``\r\n too.
  | We end with a newline, indent to old n indent depth and repeat the starting glyph to indicate the end.
  | The last character in this MULTILINE STRING is a capital A
   ``
  3
```

**Sample: ` `` ` BOLD form (pipe-guarded), backed up to minimum indent, CRLF string**

JSON (it's long so most of it omitted in favor of ...)
```json
["This....capital A", 3]
```
```
   ``\r\n
| This is a BOLD MULTILINE STRING the T in this is the first character in the string.
| Notice we only use two backticks for this type of multiline string with a starting pipe.
| Because it's guarded by a starting pipe, we can subtract 0 or any even number of indent n in the middle, or just leave it at the same indent.
| This one is backed up as far as possible, to 2 indent.  We need at least 2 indent to hold the '| ' or '/ ' depending on if we folded or not.
| Backing up the indent an odd number would be FORBIDDEN.
| It's ok if the multiline string itself contains backticks like this `` or ``\r\n or ``\n, because it's required to be guarded by a pipe so we can tell where the start is when parsing.
| Folding is ok, here's how we do it. This is the allowed way to fold in this type of multiline
/ string no matter how long the lines get.  It's totally ok to leave spaces at the end of a line in any MULTILINE
/ string not just this one and it's often desirable when folding.  The fold character is in the margin, one space back from the last character of the margin
/ just like we do folding everywhere else including tables.
| We didn't start this with a ``\n or ``, but we could have and it would be allowed.  Our ending glyph should match our starting glyph, here a ``\r\n, but if
/ it were a ``, our ending glyph would have to match it and be a `` too.
| We end with a newline, indent to old n indent depth and repeat the starting glyph to indicate the end.
| The last character in this MULTILINE STRING is a capital A
   ``\r\n
  3
```

**Sample: `` ` `` MINIMAL form (single backtick, indented, no pipe)**

JSON (it's long so most of it omitted in favor of ...)
```json
["This....capital A", 3]
```
```
   `
    This is a MINIMAL MULTILINE STRING the T in this is the first character in the string.
    Notice we only use one backtick for this type of multiline string with no starting pipe.
    It's ok if the multiline string itself contains backticks like this ` or `\r\n or `\n, because it's required to be indented normally so we can tell where the start is when parsing.
    Folding is ok, here's how we do it. This is the allowed way to fold in this type of multiline
  / string no matter how long the lines get.  It's totally ok to leave spaces at the end of a line in any MULTILINE
  / string not just this one and it's often desirable when folding.  The fold character is in the margin, one space back from the last character of the margin
  / just like we do folding everywhere else including tables.
    We didn't start this with a `\n or `\r\n, but we could have and it would be allowed.  Our ending glyph should match our starting glyph, here only a `, but if
  / it were a `\r\n, our ending glyph would have to be a `\r\n too.
    We end with a newline, indent to old n indent depth and repeat the starting glyph to indicate the end.
    The last character in this MULTILINE STRING is a capital A
   `
  3
```

**Example: multiline string with LF content, default options**
```
 `
  this is a multiline LF EOL document being displayed within a MULTILINE STRING
  string the indent spaces are not part of the string but the newlines are
  this is part of the same string
  this line is super super long and the implementor wanted to fold it even though
/  the source text didn't and allowing folding of multiline text is not the default
    this is part of the multiline string with two leading spaces as part of the string
  this is the end of the string ends with just a newline and -2 indent spaces
 `
```

**Example: multiline string with LF content, non-default options (always-mark multiline + display LF as ``\n)**
```
 ``\n
| this is a multiline LF EOL document being displayed within a MULTILINE STRING
| string the indent spaces are not part of the string but the newlines are
| this is part of the same string
| this line is super super long and the implementor wanted to fold it even though
/  the source text didn't and allowing folding of multiline text is not the default
|   this is part of the multiline string with two leading spaces as part of the string
| this is the end of the string ends with just a newline and -2 indent spaces
 ``\n
```

**Example: multiline string with CRLF content**
```
 ``\r\n
| this is a multiline CRLF EOL document being displayed within a MULTILINE STRING
| string the indent spaces are not part of the string but the newlines are
| this is part of the same string
| this line is super super long and the implementor wanted to fold it even though
/  the source text didn't and allowing folding of multiline text is not the default
|   this is part of the multiline string with two leading spaces as part of the string
| this is the end of the string ends with just a newline and -2 indent spaces
 ``\r\n
```

n is an integer indent level, always 0 or an even natural number

**Bad Example (but parsable): multiline string with no data EOL**
```
 ``
| This is probably a mistake by the user or generator, but is allowed to allow intentional visual emphasis and/or easier manual editing.
 ``
```

**Good example (intentional emphasis in the right context): Still no EOL, but the generator knows it's data and may be justified here.
```
 ``
| The generator knows what goes here and wants to emphasize it intentionally to the reader even though it does not contain an EOL.
 ``
```

 **Bad Example (Parse error): multiline string with no content at all, not even a single character - this should fail to parse with any of the three multiline string forms, should have been ""**
```
 ``
 ``
```

---

## ARRAYS AND OBJECTS

### Key-Value Pairs

Key-value pair as part of object: key colon Value space space (the space space can be the same space space as ends the string)

A really long key bare or otherwise can be folded in the same manner as a string value described above.

TJSON MUST preserve key order exactly as emitted.

TJSON also MAY preserve duplicate keys (as JSON technically permits them) - key order MUST be preserved and duplicates MAY be passed through faithfully and it is the consumer's responsibility to handle them.  Many actual platforms cannot track duplicate keys anyway, making it tough to implement and unnecessary, so it's acceptable to fall back to JSON type real-world duplicate key behavior and do something platform-dependent here.  The ideal TJSON implementation should keep duplicate keys, and keep them in the original order.  This is a should, not a must.  If your platform's normal JSON type implementation and data structures support duplicate keys, you really should preserve duplicate keys.

**Example: key fold (with ludicrously short -w, shorter than we would actually allow)**

JSON: `{"areallylongobjectkey":3}`
```
  areallylong
  / objectkey:
  / 3
```

**Example: key fold, non-default forbid-bare-keys option (with ludicrously short -w, shorter than we would actually allow)**
```
  "areallylong
  / objectkey":
  / 3
```

### Packing

Packing is pushing additional values on the same line as another value, either elements of an array, or key value pairs in an object.
Key value packing and array packing have different rules.
We forbid packing key value pairs in an object on the same line as the parent key - it's just too confusing as a key value pair is not a simple type and it almost feels like an array.  We do sometimes pack an array with simple types only on the same line as a parent key as shown below.
Arrays can sometimes pack basic types on the same line as the parent key.
Packing is optional, and it's ok to pack even if the first element doesn't pack - this is within spec - the point is for generators to use the breathing room in the spec to generate easy to read text without violating the spec.

#### Key-Value Packing Examples

**Example:** `{"A":1, "B":2, "C":3}`
```json
{"A":1, "B":2, "C":3}
```
```
  A:1    B:2    C:3
  // This is packing too, and even though they aren't strings, due to the keys, they can be safely packed with spaces, so this is ok.
```
```
  A:1
  B:2
  C:3
  // This is not packed.
```

**Example:** `{"X":{"A":1, "B":2, "C":3}}`
```json
{"X":{"A":1, "B":2, "C":3}}
```
```
  X:
    A:1
    B:2
    C:3
  // This is not packed.
```
```
  X:
    A:1    B:2    C:3
  // This is packed, and looks pretty good.
```
***Example (not allowed):***
```
  X:  A:1
    B:2    C:3
  // This is FORBIDDEN.  This is packed, and might theoretically be parseable, but is confusing because an object with kv pairs is just too complex for this type of treatment. This type of packing is still legal with arrays containing only basic types (an object key value pair is not a basic type).  In general, it's better looking not to pack on the same line as the parent key unless you are putting the whole thing on that line, but it is more compact which may be more important.
```

#### Array Packing Examples

**Example:** `[1,2,3]`
```json
[1,2,3]
```
```
  1, 2, 3
  // the above line is a packed array.  We put more than one basic value on the same line.  If it was just the first element in the array, it would be '  1'
```

**Example:** `["1", "2", "3"]`
```json
["1", "2", "3"]
```
```
   1   2   3
   // This type of packing is only allowed because all the types are strings - packing the numbers wouldn't be allowed because it would look like the strings
```


**Example:** `{"A":[1,2,3,4,5,6,7,8,9,10,11]}`
```json
{"A":[1,2,3,4,5,6,7,8,9,10,11]}
```
```
  A:  1, 2, 3, 4, 5, 6, 7, 8,
    9, 10, 11
    // this is also pretty ugly, but is more compact TJSON
```
```
  A:  1,
    2, 3,
    4,
    5, 6, 7, 8,
    9, 10, 11
// this is completely legal, though probably not a great choice for generators, unless 4 is a multidigit long number.
// Just because one line is packed doesn't mean they all need to be.
```
```
  A:
    1, 2, 3, 4,
    5, 6, 7, 8,
    9, 10, 11
  // this is better looking, but less compact than if we had put some of the numbers on the line with the bare key "A".
```

**Example:** `["ABC DEF GHI", "XYZ 123 456", "abc def ghi", "x"]`
```json
["ABC DEF GHI", "XYZ 123 456", "abc def ghi", "x"]
```
*(ok, but ugly — generator is using a packing separator with no actual packing and strings that should probably be bare instead)*
```
  "ABC DEF GHI",
  "XYZ 123 456",
  "abc def ghi",
   x
  // this is technically legal, but you aren't actually packing anywhere in the entire array as you don't have more than one value on the same line, so it's better not to use a packing separator at all.  Implementors as always can make their own aesthetic choices.
```
*(good and also legal, not packed)*
```
   ABC DEF GHI
   XYZ 123 456
   abc def ghi
   x
```
*(good example)*
```
  ABC DEF GHI   XYZ 123 456   abc def ghi   x
```
*(also a good example for small width)*
```
   ABC DEF GHI
   XYZ 123 456   abc def ghi   x
   // It's ok and looks good here to use a packing character on the first line even though there's no actual packing going on on that line because we are packing the other lines and we want to remain consistent.
```

Generators may not output bare strings containing commas when packing (double quote the strings that contain commas instead). Double quoted strings with commas are within spec in both variants but may or may not be desirable.

**Example:** `["ABC, DEF, GHI", "XYZ, 123, 456", "abc, def, ghi", "x"]`
```json
["ABC, DEF, GHI", "XYZ, 123, 456", "abc, def, ghi", "x"]
```
*(bad example, parse error — bare strings with commas used as separator for a comma packed array which forbids bare strings)*
```
   ABC, DEF, GHI,
   XYZ, 123, 456,
   abc, def, ghi,
   x
   // this is why we don't allow this.
```
*(bad example, parse error — same but packed - this is part of the reason why we forbid bare strings in comma packed array lines)*
```
   ABC, DEF, GHI,  XYZ, 123, 456,  abc, def, ghi,  x
   // this is why we don't allow this.
```
*(good example, not packed)*
```
   ABC, DEF, GHI
   XYZ, 123, 456
   abc, def, ghi
   x
   // This is a good way to handle this (it isn't packed at all, and for this kind of content, that may be the best thing you can do)
```
*(good example, packed correctly)*
```
   ABC, DEF, GHI   XYZ, 123, 456   abc, def, ghi   x
  // This is legal, and compact, and pretty readable
```
*(another good example)*
```
   ABC, DEF, GHI   XYZ, 123, 456
   abc, def, ghi   x
  // this looks pretty good too
```
*(a parseable but just ok and somewhat ugly example that some generators might prefer)*
```
  "ABC, DEF, GHI", "XYZ, 123, 456", "abc, def, ghi", "x"
  // also looks ok, but more quote laden
```

### Array Format

1) Each element gets it's own line - always allowed, this is the default
Array starter 1: (increment indent level by 2) newline + indent level spaces (parse first key to see if it's an object or not)
Array separator 1: (works for anything, the default) newline + n spaces
Array ending 1: newline + n-2 spaces, decrement indent level by 2

Note on packed array formats 2 and 3 below:
Note that we are following the rule that the first element determines the bare string/non-bare string type - this is so the user only has to look for the extra space at the beginning of the line and not otherwise to read a packed array.  Many users of TJSON will simply not know or care about types, and they are able to ignore them if they like, but we need a sophisticated reader with a basic understanding of the format to be able to recover type information without remembering more than one or two rules (bare string starts with extra space, being the main one).  This is different from prior versions of the format, but necessary both for readability and reliable parsing without behaving in a way that is mysterious to the sophisticated reader.  It's not enough to have rules that enable consistent parsing; they also need to be relatively simple so that the subset of readers that care about types can quickly determine the type reliably when they need to do that.  Preferably, we want a reader of TJSON to be able to pick up the typing rules without being told if at all possible.
We are also allowing packed array formats to be on the same line as the parent key also, though this will usually look bad unless the entire array contents can be fit on that line, but it can aid compactness and is left up to the generator.  It must parse either way.
Different lines in the representation of the same data array can pick 1), 2) or 3) without parse issues as all are fairly readable together.  This is intentional.  This includes packing a one or more array elements on the line with the parent object key of the array and using any array packing strategy for the other lines.

2) Comma separated packed array line format
BARE STRINGS ARE NOT ALLOWED:  This is because it can become parse ambiguous and/or confusing to the reader if you allow a comma that is not double quoted and not a separator.  Other rulesets are possible that would disambiguate parsing, but the rules need to be really simple in order to allow a reader that knows only one or two things about TJSON to reliably discern the type when they need to.  We also do not want to force ourselves to double quote bare strings with commas for clarity, as bare strings with commas are pretty common and avoiding unnecessary quotes for readability is one of the strengths of the format.
PARSER must error if it finds a violation of this rule.  This is important to avoid disasters like this: '  word,  word, 5,  word, 6' or '  word,  word, "word",  bare "string with" quotes'  which are hard to read and parse and probably an editing mistake.  The user can simply push it onto its own line unpacked if the user wants to add a comma somewhere.  Editing is a lower priority than reading in TJSON so this is an acceptable inconvenience for editors.
Array starter 2 (inline start variant): increment indent level by 2, space space ( '[ ' here instead of '  ' is also acceptable if the writer wants to be particularly explicit, but generators should not generate it as it's ugly, this is a level of explicitness beyond force markers)
Array starter 2 (next line start variant): just like array starter 1 (this is acceptable, but it usually looks better to just start on the next line if it doesn't all fit on one line with the key, so the default is to do that)
Array separator 2: `", "` or `",\n"` + (n+2) spaces (continuation: comma at end of line, next line indented +2 from the key's indent (which is where we would have been anyway if we used array starter 1). All continuation lines use this same fixed indent.)
The trailing ',' on a comma packed line is optional and SHOULD never be used for the last data element of an array or a line with only one element of the data array (but it should still parse for both).  Bare strings are not allowed to end with a comma, so this can both ease reading and reinforce to the knowledgeable reader that the line they are looking at does not contain a bare string.
Array ender 2 (inline variant): newline at n-2 indent, decrement indent level by 2

3) Double space separated packed array line format
REQUIRES that all elements of the array on that line are bare strings.
REQUIRES that all elements of the array on the same line be separated by 3 spaces from each other.
Array starter 3 (inline start variant): increment indent level by 2, space space ( '[ ' here instead of '  ' is also acceptable if the writer wants to be particularly explicit, but generators should not generate it as it's ugly, this is a level of explicitness beyond force markers) (will effectively be three spaces because what follows is required to be a bare string)
Array starter 3 (next line start variant): same as array starter 1 (increment indent level + 2, newline + indent level spaces)
Array separator 3: `"  "` or `\n` + n spaces (bare strings will be effectively three spaces because bare strings start with a leading space themselves).  If we do a newline, the next array elements will end up right where they would have been if we used array starter 1.  This is not an accident
Array ender 3: newline

These array packing rules are tighter than they were before v0.5.0 - if you need a prior version to generate tjson (just upgrade instead, it's more straightfoward) that can be ingested by later versions, turn off array packing with options.

You are not allowed to fold within either of the two packed array formats (Type 2 or Type 3 above).

It is absolutely acceptable and often preferable for generators to use multiple array packing formats in the same data array (this was formerly discouraged before 0.5.0, but having tighter array packing rules here prevents reader confusion without discouraging generators from mixing packing formats in the same array, as now the packing format itself helps the reader discern type information).

### Object Format

Object starter: newline + (increment indent by 2) + indent level spaces

A JSON key is always a string, which we exploit - a BARE KEY doesn't need a preceding space to distinguish it from other types because we know the next thing HAS to be a string as JSON can't have non-string keys. A `:` in a key must be shown as a JSON STRING with `"keystring:morestring"` - in other words we are using a JSON string here.  BARE KEYS are preferred to JSON STRINGS by default, but sometimes a particular key will not meet the requirements of a BARE KEY and must be a JSON STRING instead.

FOLDING is not allowed in CANONICAL TJSON

Anything else as a key name is just like JSON `"keyname"` with the same rules

### Width and Packing Settings

Two separate width settings:

- wrap/pack width (default 80, clamped to minimum 20) — controls line wrapping and maximum packing width for objects, arrays, inline values etc.  If something doesn't pack, it gets its own line, even if it's longer than the pack/wrap width.
  - Rationale:  It doesn't make sense to pack or wrap everything - sometimes it's just better to let it be longer than the width setting, particularly if it's incomprehensible gibberish anyway as it will make your editor (not the TJSON) fold it onto the very beginning of the next line in your text editor, making it take up more horizontal space, meaning it will take up less vertical space and it will be over quicker.  Alternately, your text editor may just cut it off after the width, which is ok too as it's gibberish anyway.  Also, indent levels might get beyond your wrap/pack width in some samples, in which case it's going to go past the wrap/pack width, and that's ok.  We can't make every bit of data perfectly human readable all the time.  Implementors are free to make decisions they think are reasonable regarding when and whether to enforce the wrap/pack width when producing TJSON.
- Table column max width (default 40) — controls the maximum width any column is padded to. Independent of pack width so that pack width can be set to infinity (pack more stuff onto the same line forever) while still keeping table columns sane, or vice versa.
- Do not pack within tables!  This is not allowed (both because it would be hard to read, and because we don't allow any of the things that we might pack (nonempty arrays or nonempty objects) within tables at all).  Use the normal format instead if you need to do that.  Wrapping strings within tables is strongly discouraged, implementors SHOULD not do that, but it is allowed.
- Don't both pack and wrap on the same line unless you have a very good and particular reason!  It's forbidden in tables to pack, but even in other places, both packing and wrapping on the same line is almost certainly going to look horrible.  Implementations SHOULD not both pack and wrap on the same line - they should pick one or the other for that line, or neither.  Going past our pack/wrap width is sometimes acceptable but should be avoided if possible.

### Array and Object Disambiguation

How do we know a base level array is not an object? Look at the first element - try to parse it as an object key-value pair, if it works it's an object, if not it's an array - because the object key has no preceding space unlike a bare string in an array, this is more reliable than it might seem at first.

One of the issues here is the indent level can be a bit opaque when we have nested arrays, like `[[{a:1}]]`, if the user just saw something like

*(spec counterexample — not real TJSON)*
```
      a:1
```

they would be incredibly confused.  So, when we bump up multiple levels at once, we require markers in the indent level to notify the user that there's an array nesting or object nesting level here.  So instead we do:
```
[ [ { a:1
```

We don't want to do this all the time, because for one jump at once it enhances readability to exclude it, so `{a:1}` becomes by default:
```
  a:1
```

but `[{a:2}]` becomes:
```
[ { a:2
```

it can also remove ambiguity, for instance `[{a:1, b:2},{c:3}]` vs `[{a:1}, {b:2, c:3}]` — how is the user to know which is which without the marker? so we add it:
```
[ { a:1
    b:2
  { c:3
```
vs
```
[ { a:1
  { b:2
    c:3
```

without the array, the user can be sure they are all the same, so it's more obviously one object: `{a:1, b:2, c:3}`
```
  a:1
  b:2
  c:3
```

optionally (not default!) we can force inclusion of these markers even for a single level, like: `{a:1, b:2, c:3}`.  This is not the default and should not be - reason being that it creates more glyphs for the user's eye to follow but it's allowed only because in certain contexts the user is not looking at enough of it at once to get the whole picture, and I think it might help - I can see this making sense when the user can only see a few lines at a time for instance, as they know they are seeing the start of the object without having to read one line up - but this is a corner case and should not be the default.
```
{ a:1
  b:2
  c:3
```

### Nesting Markers

NESTING MARKERS - (sadly necessary in some cases to make the data round trip, and to distinguish between an array of objects and an array with one object)
Nesting markers do not apply within MINIMAL JSON which is treated as a single unfoldable value and has no markers of any kind.

A nonempty array or nonempty object may be introduced with explicit nesting markers. `[` means the element on that line is part of a nonempty array.  `{` means the element on that line is part of a nonempty object. Each marker is followed by exactly one space, and sits inside the indent.  Multiple markers may be chained on the same physical line.  After the final marker, the rest of that line contains only the first child of the deepest nested container. Additional children of that container continue on later lines at the normal deeper indent.  This syntax should only be used for nesting where we would otherwise jump more than one n+2 indent levels at once (though not as a default, implementations may allow it to be forced for just one n+2 indent level by the user with an option).  Empty arrays and empty objects are written exclusively as `[]` and `{}`. Ordinary nonempty arrays and objects remain implicit unless the first physical line already uses one or more of these explicit nesting markers; in that case the outermost container on that line is explicit too.  The point of this is to be stealthy as possible, but by the time we are nesting this sort of stuff it's already looking like code, so we might as well be explicit with the very first level too.  It is allowed, but not required and should not be default, to include `[` and `{` nesting markers even when there's only one level.

OPTIONAL NESTING MARKERS: Programs that produce TJSON should by default apply optional nesting markers consistently in different parts of the output, but programs that parse TJSON should allow optional nesting markers to be applied on some lines but not others without failing to parse.  It is an error to inconsistently print optional nesting markers within the same output line.  A generator must print all of the optional nesting markers within the same output line, or none of the optional nesting markers within that output line; printing only some of the optional nesting markers on a line is forbidden, and a parser is allowed to reject such input.

THE CRITICALITY OF PRECISE SPACING AND WHY MARKERS ARE REQUIRED FOR MULTI-LEVEL JUMPS: Nesting markers are not optional when jumping more than one level at once — they are required for two reasons. First, without them the reader cannot know how many levels of nesting they are looking at. Second, and more subtly, the absence of a `{` marker is itself meaningful: it tells the reader and parser that the content on that line is not an object key-value pair. For example, `[ [  key: value` contains the bare string "key: value" — the reader knows it is a bare string precisely because there is no `{` before it. If markers were optional at multi-level jumps, this disambiguation would be lost. A line like `    key: value` at n=4 with no markers is a parse error, not because the content is wrong, but because the structural path to get there has not been defined. The wrong number of indent spaces is similarly a hard parse error and not something a parser should attempt to recover from — both cases preserve future design surface and keep the format unambiguously parsable.  Also, the human eye is much better at telling the difference between 'zerospace' 'one space' 'two  spaces' and 'three   spaces' than it is at telling the difference between 'four    spaces' and 'five     spaces', so I have attempted to make sure that anything over three spaces (and over two where possible) is never meaningful to the reader.  The reader is being made to become accustomed to an expected cadence that they will almost subconsciously be able to pick up without being told what is going on.  That completely breaks if you start expecting the reader to see the difference between five and six spaces, or just adding extra space somewhere because you don't see the reason not to.  As such spacing errors MUST be a hard failure.

### Nesting Examples

**Example:** `[[1]]`
```json
[[1]]
```
```
[ [ 1
```

**Example:** `[[[1]]]`
```json
[[[1]]]
```
```
[ [ [ 1
```

**Example:** `[{"a":1}]`
```json
[{"a":1}]
```
```
[ { a:1
```

**Example:** `[[1],2]`
```json
[[1],2]
```
```
[ [ 1
  2
```

**Example:** `[[1,2]]` (canonical, no packing)
```json
[[1,2]]
```
```
[ [ 1
    2
```

**Example:** `[[1,2]]` (with packing, default)
```
[ [ 1, 2
```

**Example:** `{"x":[[1]]}`
```json
{"x":[[1]]}
```
```
  x:
  [ [ 1
```

**Example:** `{"x":[{"a":1},{"b":2}]}`
```json
{"x":[{"a":1},{"b":2}]}
```
```
  x:
  [ { a:1
    { b:2
```

**Example:** `[6, "6:5", "6:5"]`
```json
[6, "6:5", "6:5"]
```
```
  6
  "6:5"
   6:5
```

**Example:** `{"6":6, "6":"6:5", "6":"5"}`
```json
{"6":6, "6":"6:5", "6":"5"}
```
```
  6:6
  6:"6:5"
  "6": 5
```

**Example:** `{"6":6, "6":"6:5", "6":"5"}` (optional, usually the default though)
```
  6:6  6:"6:5"  "6": 5
```

**Example:** `{"a": 5, "6": "fred", "xy": [], "de": {}, "e": [1]}` (not default, diff-friendly, preferred for canonical TJSON)
```json
{"a": 5, "6": "fred", "xy": [], "de": {}, "e": [1]}
```
```
  a:5
  6: fred
  xy:[]
  de:{}
  e:
    1
```

**Example:** same JSON with optional not-default forced single-layer marker (not part of canonical TJSON)
```
{ a:5
  6: fred
  xy:[]
  de:{}
  e:
  [ 1
```

**Example:** `{"a": 5, "6": "fred", "obj": {"o1": 8, "o2": ["ary1veryverylong", "ary2veryverylong", "ary3", "ary4", "ary5", "ary6medium", "ary7"], "o3": "o3value"}}` (all array elements are strings, so we can but are not required to pack them together)
```json
{"a": 5, "6": "fred", "obj": {"o1": 8, "o2": ["ary1veryverylong", "ary2veryverylong", "ary3", "ary4", "ary5", "ary6medium", "ary7"], "o3": "o3value"}}
```
```
  a:5
  6: fred
  obj:
    o1:8
    o2:
       ary1veryverylong
       ary2veryverylong
       ary3   ary4   ary5
       ary6medium   ary7
    o3: o3value
```

### KEY-VALUE PACKING

WARNING:  Key value packing spacing other than 4 is experimental and subject to change.  If it changes it's going to get less permissive, not more permissive than listed below.

As is rarely the case in TJSON, there is exactly one type after every value, a key (always a string, usually expressed as a bare key).  Parsing does not require tightly policing the number of spaces between a key: value pair and the next key: value pair on the same line.  Still, visual perception demands some restrictions.

We default to 4 spaces, but we allow any even number of spaces that is at least two.  Why only even numbers?  Well we might want to parse odd numbers > 2 for ease of editing, but we should never produce them.  Three spaces looks too much like space packing of bare strings, or an array with an initial element that is a bare string - not that it's logically the same of course, but we want to reserve that space in the user's brain that three space look for only certain things.  So a generator should NEVER produce 3 spaces between packed key:value pairs.  A parser is allowed to accept three, but isn't required to.  The reason a parser might want to accept three is because a hand edit might make that mistake and beyond at least two, it isn't really something we have to enforce to ensure parsability.  If a user does that, and a parser chooses to accept it, and tries to write something out, it should NEVER try to preserve the 3 spaces.  Similarly, if a parser decides to accept less spaces than two in a corner case where it doesn't create parse ambiguity, it should never try to preserve the incorrect spacing.

A higher odd number of spaces (5 or more) are far less dangerous, but still damages the visual cadence of TJSON with no real benefit, so we are strongly recommending that generators not produce odd spacings at five or above.  If you have a specific generator with a specific reason, an attempt to align keys on different rows for example, or a very high odd spacing (7, 9, 11, 13, etc.) meant to right justify a value, it might make sense to go against this recommendation.  The entire point is to allow generators opportunities to make things more readable, and aligning keys on various lines is a good reason, but don't go below 4 spaces between key value pairs to do it.

Zero or one space between kv pairs is usually going to be ambiguous to parse as the value is often a bare string, so we have to forbid generating anything under 2 in all circumstances.  We should never successfully parse anything under two spaces here, even if the previous value would allow it without ambiguity.

Four spaces is a better default than two because two spaces are used for so many other things, and it makes many kv pairs on a line much easier to read.  After some time, it will subconciously indicate packed kv pairs to the user.

**Example: Key-value packing (kvPackMultiple 2, 4 spaces, the default)**
```
  key:   value1 with space   value2 with space   value3 with space
  key2arrayofnumbers:  1, 2, 3, 4, 5, 6, 7, 8
  key3nestedobject4space:
    nestedkey: value4    nestedkey2: value5    nestedkey3: value6
    // line above is easier to read kv pack separation of 4 spaces
```

**Example: Key-value packing (kvPackMultiple 1, 2 spaces, not the default)**
```
  key:   value1 with space   value2 with space   value3 with space
  key2arrayofnumbers:  1, 2, 3, 4, 5, 6, 7, 8
  key3nestedobject2space:
    nestedkey: value4  nestedkey2: value5  nestedkey3: value6
    // line above is parsable kv pack separation of 2 spaces, but tougher to read
```

**Example: Key-value packing (kvPackMultiple 2, 4 spaces, the default, but with extra spaces opportunistically added by the generator for alignment after Bob)**
```
  // note that we have extra spaces between Bob and age, and that we do not go below 4 spaces in between.  This is a good reason to break the spacing recommendations above.
  // we can't make this a table because of different keys, but a generator is allowed to add extra spaces in between to align it.  This is easy to parse and looks good.
  // The keys are intentionally different to prevent table rendering in this example.
  team:
  [ { namea: Alice    agea:30    rolea: admin
    { nameb: Bob      ageb:25    roleb: user
    { namec: Carol    agec:35    rolec: user
```

**Example: multiline string and inline packing (width = 40 (default is 80), multiline folding is off (the default), we are preferring commas for packing arrays over spaces (the default), string folding auto (the default), and kvPackMultiple 2 (default is 2, this means four spaces between packed key values))**
```
// this first line is a key value pack with a kv spacing of 4 (kv pack multiple of 2, the default)
  a:5    6: fred
  obj:
    o1:8
    o2:
       ary1
       `
        this is a multiline
        string it has two additional indent spaces the indent spaces are not part of the string but the newlines are part of this string and the first letter
        of the multiline is the t in "this is a multiline".  I can even use backticks in here like `this` it's totally fine even at the beginning of a line like
        `this` because it's at the wrong indent level to start and end it.  If a text looks too likely to fake out the user the implementor can choose `` instead if they like.  It parses fine for the same reason, no matter what is in here.
        this is part of the same string
        MULTILINE STRINGS SHOULD contain at least one real unescaped newline, but MUST contain at least one character in order to parse.
          this is part of the multiline string with two leading spaces as part of the string
        this is the end of the string the entire multiline ends with just a newline that's not part of the string and -2 indent spaces and a ` at
        n+1, sort of like a bare string ` but a bare string can't be a ` so the user knows it's not a bare string, it's the end of the multiline.
        The last character of this multiline string is not an EOL it is an exclamation point!
       `
       ary3   ary4
      // ary3 and ary4 are packed with each other, they are both siblings of ary1 and the multiline
    // here o3stringvalue is a basic value on o3, not an element of an array, so it must start on the same line
    o3: o3stringvalue
    // Note that we can't fit everything onto the line with o4:, so we start on the next line.  This is a rendering choice but I think it usually looks the best.
    o4:
       arraystring1,  arraystring2,
       arraystring3, 18,  arraystring4
    // here we can fit everything on the same line with o5, without running into width, so we do.
    o5:   arraystring5,  arraystring6
  obj2usinginlineformatsometimes:
    o1: str1    o2: str2    o3:-9    o4:10e3
    // the line above is a key value pack with a kv spacing of 4 (kv pack multiple of 2, the default)
    o5: this one is super duper long it
    /  is a basic value a bare string
    /  folding it is our only option it
    /  is on its own line and it still
    /  doesn't fit and because we can't
    /  pack it with anything else it
    /  must be alone.
    o6: longerstr3withitsownline
    o7: longerstr4withitsownline
    // this next line is a key value pack, with kv spacing of 4 (kv pack multiple of 2, the default)
    o8: str5    o9: str6    o10:-18
```

### Basic Examples

**Example:** `[]`
```json
[]
```
```
[]
```

**Example:** `null`
```json
null
```
```
null
```

**Example:** `"a"`
```json
"a"
```
```
 a
```

**Example:** `5`
```json
5
```
```
5
```

**Example:** `{}`
```json
{}
```
```
{}
```

**Example:** `["a",6,true,false,null,"abc","it-s;:complicated","contains, comma",[],{}]` (multiline, default)
```json
["a",6,true,false,null,"abc","it-s;:complicated","contains, comma",[],{}]
```
```
   a
  6
  true
  false
  null
   abc
   it-s;:complicated
   contains, comma
  []
  {}
```
**Example:** same JSON (optional, packed; generator chose to split it between lines, chose not to use a trailing comma on the first line)
```
  "a", 6, true, false, null, "abc"
   it-s;:complicated   contains, comma
  [], {}
```
**Example:** same JSON (optional, packed; generator chose to put it all on one line)
```
  "a", 6, true, false, null, "abc", "it-s;:complicated", "contains, comma", [], {}
```
**Example:** same JSON (optional, packed; generator chose to put the string with a comma in it in double quotes for type clarity, or maybe to fit a fixed width so it can pack the next line; generator also chose to use a trailing comma on the first comma packed line)
```
  "a", 6, true, false, null,
   abc   it-s;:complicated
  "contains, comma", [], {}
```
**Bad Example, Parse Error:** same JSON (illegal attempt to comma pack a bare string)
```
  "a", 6, true, false, null, abc
   it-s;:complicated
  "contains, comma", [], {}
```
**Bad Example, Parse Error:** same JSON (illegal attempt to add a optional trailing pack comma a the bare string it-s;:complicated, even if nothing else is on the line)
```
  "a", 6, true, false, null, abc
   it-s;:complicated,
  "contains, comma", [], {}
```
---

## TABLES

### Array of Object Table Format

Array of object TABLE format is useful for tables where every element has similar keys in consistent orders and the lengths are reasonable.  Table format is NOT CANONICAL - but it is allowed and should usually be preferred by default if all the values are not nonempty objects and not nonempty arrays (all other values are allowed), and they all have keys that are compatible enough to make sense to have in a table.  If the objects don't have their keys in consistent orders, table format is NOT ALLOWED as it could lead to key order data loss when they are read back out.

A table format should not be used to generate a row that represents an empty object.  That would violate the rule that there is only one representation of an empty object, and it would likely be confusing to the user.  However, it should parse anyway even though it should never be generated.  The purpose of this is to give a little bit more flexibility for specialized generators.

Don't use table format if it doesn't make logical sense for your use case - the point is to be readable.  If you have three objects with three keys each and no keys in common, sure, you COULD create a table out of it, but it would make it less readable not more, so don't do it.  The point is to be readable.

Actual programs that create TJSON may have options to suggest when tables are good or bad for your use case, such as a minimum number of rows or columns, a threshold of object commonality that must be reached before making a table, or possibly even some sort of a hinting mechanism to tell the function which exact things should use Array of Object format - perhaps even pointers to an object that can be compared on traversal, or some sort of selector system, but that's outside of the scope of this spec.

The header line starts with `|` at the object indent level, followed by column key names separated by `|`, with a mandatory closing `|`.  Each data row contains the values for those keys in order, separated by `|`.  Each value uses normal TJSON type-tag encoding: strings need a leading space, numbers/booleans/null do not.  It's important that they are mandatory because a mistaken edit will usually change the effective number of non-data `|` characters in a line, but that only works if they are all mandatory and trigger a parse error when we count the wrong number.  Both too few and too many `|` characters must be a parse error for that reason.

A table line can be thought of as a special way of representing an object in an array where all the objects in the array follow certain rules.  Where an indent marker is required for a table, it should be `[ ` in the indent immediately before the `|firstkey  ` (the start of the first line with the keys), with the `[ ` in the indent at the indent level of the array because the table is an array of specially rendered objects.

Tables may not share a line with anything other than indent (indent markers are by definition part of the indent, there may be indent markers on the first line).

Two pipes in a row here `||` with nothing in between or only spaces in between just means that object didn't have that particular key, if it does and you want an empty string, `""` is the only way to represent an empty string here as is the case everywhere else in the spec.  This logically implies that an absent cell means the key is not present in that object, for example, null means present with a null value, and `""` would mean present with an empty string value.

Padding: each cell is padded with spaces on the RIGHT only, minimum 2 spaces of right padding on bare strings (which have one space at the beginning built in), with the goal of aligning columns as much as possible.  The effective right delimiter for each cell is always `  |` (at least two spaces then pipe) — the parser strips trailing spaces from each cell before interpreting the value.  The first `|` on the header line aligns with the data rows (both at indent n+2).

An absent cell (key not present in that object) is represented by an empty cell - just padding spaces between the `|` separators, or no space at all between the two separators.  An explicit empty string value is represented as `""` just like it is everywhere else in TJSON.

Uneven and ugly padding is not desirable, and generators should mostly not produce tables if a reasonable looking table cannot be produced, but parsers must accept ugly and uneven padding too as long as it complies with the rules.  Our examples generally have very nice perfectly even padding, but nice padding is not required for parse validity.  Padding does not have to be consistent from row to row or column to column to be valid.  The two space minimum on the right side after a value is required, but not every cell will necessarily contain a value, and a cell without a value can go as low as zero width and still parse.

**Example:** table format
```json
[{"a":1, "b":"xyz", "c":"def", "d":"yyy", "E":5},{"a":"12", "b":14, "c":null, "d":"yxx"},{"a":true, "b":14, "E":"yxx"}]
```
```
  |a        |b        |c        |d        |E     |
  |1        | xyz     | def     | yyy     |5     |
  | 12      |14       |null     | yxx     |      |
  |true     |14       |         |         | yxx  |
```

*(HORRIBLE, INCONSISTENT, UGLY, DO NOT PRODUCE THIS, BUT TECHNICALLY PARSABLE AND VALID example data equivalent to the above)*
```
  |a                               |b        |c        |d        |E     |
  |1   | xyz    | def                         | yyy     |5     |
  | 12      |14       |null     | yxx  ||
  |true     |14                                |         |         | yxx  |
```

### Folding Within a Table  **Experimental and Optional - Not fully implemented, and subject to change, may never be implemented**

FOLDING WITHIN A TABLE - This is turned off by default, and is almost always a bad idea.  Folding in a table is just like folding other places.

Logically, a table row is a special case way of rendering an entire object at once.  As such, if a line is folded, it should be folded at the level of the object itself, not the members of the object, or the surrounding array.

A table may also fold lines, and if it does it will have a fold marker on the folded lines.  The fold marker `/ ` aligns with the opening `|` on each line.  This works very much like a fold of an object key with an inline simple value.  (The example below helps a lot to understand this one.)

In non table contexts, there is a guaranteed space after the fold marker because it is placed within the margin, and we also have a non-data space after the `/` here.  The reference implementation doesn't generate folded table lines by default, and other implementations do not have to generate this, but all implementations do have to correctly parse it.

Folding is allowed only within optional extra right padding (probably better looking) and between the first data character (this would be the a in both the JSON double quoted `|"abc"` and the bare string `| abc`) and the last data character of the string that follows the table `|`.  This is exactly like the rules for string folding intentionally.  Number folding follows the same rule.

Folding does not change the rules as far as empty cells or the exact number of `|` characters.

If there is no value in that field, you can fold between two pipes.  This is not an endorsement that generators should do this, only a statement of what is parsable and allowed.

If you want to fold in a table, it's usually better to let the line run long past your desired output width, or not use a table at all.  But it is allowed and parsers should accept it.

**Example: table fold (moderately ugly, but valid; someone with a generator that absolutely must not exceed 50 chars wide — do not do this by default)**
```json
[{"a":1, "b":"xyz", "c":"def", "d":"yyy", "E":5},{"a":"12", "b":14, "c":null, "d":"yxx REALLYGOTTAHAVEITABSOLUTELYMUSTFOLDFORSOMEREASON"},{"a":true, "b":14, "E":"yxx"}]
```
```
  |a        |b        |c        |d        |E     |
  |1        | xyz     | def     | yyy     |5     |
  | 12      |14       |null     | yxx REALLYGOTTAH
  / AVEITABSOLUTELYMUSTFOLDFORSOMEREASON  ||
  |true     |14       |         |         | yxx  |
```

*(Forbidden — fold between `|` and the start of the value; parsers do not have to accept either of these forbidden folds; there is nothing else unparsable or forbidden about the example, though it is ugly)*
```
  |a        |b        |c        |d        |E     |
  |1        | xyz     | def     |
  / yyy     |5     |
  | 12      |14       |null     |
  /  yxx REALLYGOTTAH
  / AVEITABSOLUTELYMUSTFOLDFORSOMEREASON     |
  /       |
  |true     |14       |         |         | yxx  |
```

**Example: if you are going to fold anyway, this is probably the way to do it**
```json
[{"a":1, "b":"xyz", "c":"def", "d":"yyy", "E":5},{"a":"12", "b":14, "c":null, "d":"yxx"},{"a":true, "b":14, "E":"yxx THIS IS REALLY LONG"}]
```
*(width = 52, not the default) (and we wouldn't fold at all in a table by default either)*
```
  |a        |b        |c        |d        |E       |
  |1        | xyz     | def     | yyy     |5       |
  | 12      |14       |null     | yxx     |        |
  |true     |14       |         |         | yxx THIS
  /  IS REALLY LONG  |
```

*(With forced markers on, not the default) (width = 52, not the default) (and we wouldn't fold at all in a table by default either)*
```
[ |a        |b        |c        |d        |E       |
  |1        | xyz     | def     | yyy     |5       |
  | 12      |14       |null     | yxx     |        |
  |true     |14       |         |         |"yxx THIS
  /  IS REALLY LONG AND CONTAINS A PIPE | SO WE MUST
  /  DOUBLE QUOTE IT FOLDED OR NOT BECAUSE IT IS IN
  /  A TABLE"  |
```

**Example: folding the header row is valid, but not default and probably a really bad idea**
```json
[{"a":1, "b":"xyz", "c":"def", "d":"yyy", "E THIS IS REALLY LONG":5},{"a":"12", "b":14, "c":null, "d":"yxx"},{"a":true, "b":14, "E THIS IS REALLY LONG":"yxx THIS IS REALLY LONG"}]
```
*(width = 52, not the default, force indent markers on, also not the default) (and we wouldn't fold at all in a table by default either)*
```
[ |a        |b        |c        |d        |E THIS
  /  IS REALLY LONG      |
  |1        | xyz     | def     | yyy     |5       |
  | 12      |14       |null     | yxx     |        |
  |true     |14       |         |         | yxx THIS
  /  IS REALLY LONG  |
```

**Example: not table-compatible (would lose key order if stored as a table and round-tripped — must use non-table display)**
```json
[{"a":1,"b":2},{"b":1,"a":2}]
```
TJSON is not allowed to render this as a table irrespective of options as it can't do it without losing key order data, so we must use non-table rendering instead:
```
[ { a:1
    b:2
  { b:1
    a:2
```

**Example: stacking a table** (this table is too small for default settings to make it a table, but used here as an example; a larger table would work the same way)
```json
[[[[{"a":"apple","b":2},{"a":3,"b":"pear"}]]]]
```
*(this is not with forced markers on, but they are required here due to the multilevel jump at once from the nesting)*
```
[ [ [ [ |a       |b      |
        | apple  |2      |
        |3       | pear  |
```

**Example: nested object with table (rendered as table — not the default for this size, and with a fold — definitely not the default; used to illustrate fold syntax)**
```json
{"level1":{"level2":{"level3":[{"a":"apple","b":2},{"a":3,"b":"pearfoldnospaces"}]}}}
```
*(width = 23, folding within a table is not default behavior)*
```
  level1:
    level2:
      level3:
        |a       |b     |
        | apple  |2     |
        |3       | pearfo
        / ldnospace  |
```

*(width = 25, non-default option force markers is on, folding within a table is not default behavior)*
```
{ level1:
  { level2:
    { level3:
      [ |a       |b     |
        | apple  |2     |
        |3       | pearfo
        / ldnospaces  |
```

**Example: same JSON, not rendered as a table (default behavior for an array of structs of this size, with non-default fold for illustration)**
*(width = 25)*
```
level1:
  level2:
    level3:
    [ { a: apple
        b:2
      { a:3
        b: pearfoldnospac
        / es
```
#### Parallelism between table and non-table rendering of the same data

Note that the innermost object first data character above, the 'a' key, is in column 9, and the table representation has the first data character, the `|`, at column 9 too.The `|` in a table is effectively a data character, it isn't inside the margins like the other pipes in TJSON.  The idea is to be consistent between the table and non-table representation.  Bare strings can't start with `|` precisely so that `|` at any value position is always unambiguously a table row. The "forbidden start characters" for bare strings aren't arbitrary restrictions, they're all load-bearing in some way, either for the human viewer, or for the parsing grammar. `/` can't start a bare string because it's a fold/comment marker, `` ` `` can't because it's multiline, `|` can't because it's a table row.  The whole thing is designed so the first character tells you exactly what you're looking at, both for easy reading, and unambiguous parsing.  The reason we exclude quotelike, commalike, pipelike, as opposed to just the thing itself, is to protect humans from confusion, not the parser.  I don't want parsers to parse the quotelike but not quote and so forth cases, they have to reject them, because those inclusions while helping humans, also create design space for future feature expansion - so it's not 100% for human clarity.  I try to be clear about when something is parsable, and when it's FORBIDDEN or NOT ALLOWED.  If it's FORBIDDEN or NOT ALLOWED, and the text doesn't specifically say we should parse it even if it's wrong, we need to fail rather than parse it to preserve future design surface for features.  The same thing is true for the wrong number of spaces - that's not a snafu the parser should try to fix, it's breakage that ought to cause a parsing failure.


**Example: two tables stacked in an array**
```json
[[[[{"a":"apple","b":2},{"a":3,"b":"pear"}],[{"a":"apple","b":2},{"a":3,"b":"pear"}]]]]
```
*(not with forced markers on, but they are required here)*
```
[ [ [ [ |a       |b      |
        | apple  |2      |
        |3       | pear  |
      [ |a       |b      |
        | apple  |2      |
        |3       | pear  |
```

**Example: parsable, but useless and ugly**
*(Bad example — should parse, but do not generate empty object rows unless you have a very special case generator; violates one way to represent empty object)*
```json
[{},{},{}]
```
```
  |key1  |key2  |key3  |
  |      |      |      |
  |      |      |      |
  |      |      |      |
```

*(Bad example — should parse, but do not generate empty object rows unless you have a very special case generator; violates one way to represent an empty array)*
```json
[]
```
```
  |key1  |key2  |key3  |
```

### Implementation Note — When to Use Table Format

The parser must accept any valid table format regardless of how it was generated. The following is guidance for generators only.  Note that bare strings need exactly one space before, and two or more spaces afterward.  Other data types must start immediately at the left with no space, and may but are not required to have space afterward.  Bare strings should not be allowed to contain pipes or pipelike characters in a table by default.  Bare strings by default should not be used in a table if they are exactly the string version of one of our basic types: "true", "false", or "null".  "[]" and "{}" are already illegal as bare strings generally.  If one of the strings in the table contains a pipe or pipelike character, that string must be a JSON STRING instead.  Implementations on platforms that simply do not have a basic type may choose to default to bare strings for those basic types as it is unlikely that the string "true" could be ambiguous in an environment that doesn't have a boolean data type at all.

### Default Algorithm for Deciding Whether to Use Table Format

(Your use case might need a different default depending on what your program is doing, and that's ok):

- At least 3 data rows
- At least 3 columns
- Key overlap at least 80% (average row has values for at least 80% of the columns)

If all three are met, use table format. Otherwise use regular array of objects format. Callers will typically want options or hinting to override this.

### Default Algorithm for Column Widths

- Column width = max(header key width, widest data cell value width) + 2 (minimum padding)
- If total table width fits within table column max width (default 40 characters per column), use ideal widths
- If not, distribute available space to columns that are below the average width, pushing toward equal widths, left to right. Columns already at or above the average get minimum 2 spaces only.
- Never truncate any value. A cell wider than table column max width just makes a wider column — it simply receives no bonus padding from the equalization pass.
- Width here is meant to optimize around human visibility, generators can make their own decisions, but those decisions on what a width is should at least look toward approximating visible width first.  This document is written in ascii where one byte = one character and all characters are the same size as we are using a monospace font.  In a lot of situations, TJSON might be written with a non-monospace font for all sorts of reasons, perhaps embedding it in another document, and perhaps in a font unforseeable by the generator.  A generator might keep one character = 1 width to stay simple, but it might also have a much more complex algorithm resulting in a better visual approximation for the characters used and the expected font.  A generator can have a better default algorithm than this one, it wouldn't be a violation of the specification, this should be seen as a reasonable default monospace width algorithm where each character takes exactly one space, that implementors can derive visual priority from.

### Pipelike Character Definition

A PIPELIKE CHARACTER is a vertical line (U+007C), or any other character in the following set: U+007C, U+00A6, U+01C0, U+01C1, U+05C0, U+16C1, U+2016, U+2223, U+2225, U+23D0, U+2502, U+2503, U+2506, U+2507, U+250A, U+250B, U+254E, U+254F, U+2551, U+258F, U+2595, U+2758, U+2759, U+275A, U+2980, U+2AF4, U+2AFC, U+2AFE, U+2AFF, U+2D4F, U+FE31, U+FE33, U+FF5C, U+FFE4, U+1FB70, U+1FB71, U+1FB72, U+1FB73, U+1FB74, U+1FB75

(Note: the only part of this that is currently theoretically important for parsing is `|` itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

These include both things that look like a pipe, but also things that might look like a vertical boundary because we are using pipe as a table boundary in some cases.

U+007C  |   VERTICAL LINE
U+00A6  ¦   BROKEN BAR
U+01C0  ǀ   LATIN LETTER DENTAL CLICK
U+01C1  ǁ   LATIN LETTER LATERAL CLICK
U+05C0  ׀   HEBREW PUNCTUATION PASEQ
U+16C1  ᛁ   RUNIC LETTER ISAZ IS ISS I
U+2016  ‖   DOUBLE VERTICAL LINE
U+2223  ∣   DIVIDES
U+2225  ∥   PARALLEL TO
U+23D0  ⏐   VERTICAL LINE EXTENSION
U+2502  │   BOX DRAWINGS LIGHT VERTICAL
U+2503  ┃   BOX DRAWINGS HEAVY VERTICAL
U+2506  ┆   BOX DRAWINGS LIGHT TRIPLE DASH VERTICAL
U+2507  ┇   BOX DRAWINGS HEAVY TRIPLE DASH VERTICAL
U+250A  ┊   BOX DRAWINGS LIGHT QUADRUPLE DASH VERTICAL
U+250B  ┋   BOX DRAWINGS HEAVY QUADRUPLE DASH VERTICAL
U+254E  ╎   BOX DRAWINGS LIGHT DOUBLE DASH VERTICAL
U+254F  ╏   BOX DRAWINGS HEAVY DOUBLE DASH VERTICAL
U+2551  ║   BOX DRAWINGS DOUBLE VERTICAL
U+258F  ▏   LEFT ONE EIGHTH BLOCK
U+2595  ▕   RIGHT ONE EIGHTH BLOCK
U+2758  ❘   LIGHT VERTICAL BAR
U+2759  ❙   MEDIUM VERTICAL BAR
U+275A  ❚   HEAVY VERTICAL BAR
U+2980  ⦀   TRIPLE VERTICAL BAR DELIMITER
U+2AF4  ⫴   TRIPLE VERTICAL BAR BINARY RELATION
U+2AFC  ⫼   LARGE TRIPLE VERTICAL BAR OPERATOR
U+2AFE  ⫾   WHITE VERTICAL BAR
U+2AFF  ⫿   N-ARY WHITE VERTICAL BAR
U+2D4F  ⵏ   TIFINAGH LETTER YAN
U+FE31  ︱  PRESENTATION FORM FOR VERTICAL EM DASH
U+FE33  ︳  PRESENTATION FORM FOR VERTICAL LOW LINE
U+FF5C  ｜  FULLWIDTH VERTICAL LINE
U+FFE4  ￤  FULLWIDTH BROKEN BAR
U+1FB70  🭰  VERTICAL ONE EIGHTH BLOCK-2
U+1FB71  🭱  VERTICAL ONE EIGHTH BLOCK-3
U+1FB72  🭲  VERTICAL ONE EIGHTH BLOCK-4
U+1FB73  🭳  VERTICAL ONE EIGHTH BLOCK-5
U+1FB74  🭴  VERTICAL ONE EIGHTH BLOCK-6
U+1FB75  🭵  VERTICAL ONE EIGHTH BLOCK-7

### Quotelike Character Definition

A QUOTELIKE CHARACTER is a backtick (U+0060), a double quote (U+0022), single quote (U+0027), or any other character in the following set: [\p{Quotation_Mark}`]: U+0022, U+0027, U+0060, U+00AB, U+00BB, U+2018, U+2019, U+201A, U+201B, U+201C, U+201D, U+201E, U+201F, U+2039, U+203A, U+2E42, U+300C, U+300D, U+300E, U+300F, U+301D, U+301E, U+301F, U+FE41, U+FE42, U+FE43, U+FE44, U+FF02, U+FF07, U+FF62, U+FF63

(Note: the only part of these that are currently theoretically important for parsing is `"` and `\`` itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

This includes not only things that might look like a double quote, but things that might be interpreted like a quote to the reader in a meaning sense, if not a visual one.  This is broader than these other definitions intentionally.

U+0022  "   QUOTATION MARK
U+0027  '   APOSTROPHE
U+0060  `   GRAVE ACCENT
U+00AB  «   LEFT-POINTING DOUBLE ANGLE QUOTATION MARK
U+00BB  »   RIGHT-POINTING DOUBLE ANGLE QUOTATION MARK
U+2018  ‘   LEFT SINGLE QUOTATION MARK
U+2019  ’   RIGHT SINGLE QUOTATION MARK
U+201A  ‚   SINGLE LOW-9 QUOTATION MARK
U+201B  ‛   SINGLE HIGH-REVERSED-9 QUOTATION MARK
U+201C  “   LEFT DOUBLE QUOTATION MARK
U+201D  ”   RIGHT DOUBLE QUOTATION MARK
U+201E  „   DOUBLE LOW-9 QUOTATION MARK
U+201F  ‟   DOUBLE HIGH-REVERSED-9 QUOTATION MARK
U+2039  ‹   SINGLE LEFT-POINTING ANGLE QUOTATION MARK
U+203A  ›   SINGLE RIGHT-POINTING ANGLE QUOTATION MARK
U+2E42  ⹂   DOUBLE LOW-REVERSED-9 QUOTATION MARK
U+300C  「  LEFT CORNER BRACKET
U+300D  」  RIGHT CORNER BRACKET
U+300E  『  LEFT WHITE CORNER BRACKET
U+300F  』  RIGHT WHITE CORNER BRACKET
U+301D  〝  REVERSED DOUBLE PRIME QUOTATION MARK
U+301E  〞  DOUBLE PRIME QUOTATION MARK
U+301F  〟  LOW DOUBLE PRIME QUOTATION MARK
U+FE41  ﹁  PRESENTATION FORM FOR VERTICAL LEFT CORNER BRACKET
U+FE42  ﹂  PRESENTATION FORM FOR VERTICAL RIGHT CORNER BRACKET
U+FE43  ﹃  PRESENTATION FORM FOR VERTICAL LEFT WHITE CORNER BRACKET
U+FE44  ﹄  PRESENTATION FORM FOR VERTICAL RIGHT WHITE CORNER BRACKET
U+FF02  ＂  FULLWIDTH QUOTATION MARK
U+FF07  ＇  FULLWIDTH APOSTROPHE
U+FF62  ｢   HALFWIDTH LEFT CORNER BRACKET
U+FF63  ｣   HALFWIDTH RIGHT CORNER BRACKET

### Commalike Character Definition

A COMMALIKE CHARACTER is a comma (U+002C), or any other character in the following set: U+002C, U+02BB, U+02BC, U+02BD, U+060C, U+066B, U+201A, U+2E32, U+2E34, U+2E41, U+2E4C, U+3001, U+FE50, U+FE51, U+FF0C, U+FF64

(Note: the only part of this that is currently theoretically necessary for parsing is `,` itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

This list includes things that could be visually mistaken for a comma in order to avoid confusion by the reader.

U+002C  ,   COMMA
U+02BB  ʻ   MODIFIER LETTER TURNED COMMA
U+02BC  ʼ   MODIFIER LETTER APOSTROPHE
U+02BD  ʽ   MODIFIER LETTER REVERSED COMMA
U+060C  ،   ARABIC COMMA
U+066B  ٫   ARABIC DECIMAL SEPARATOR
U+201A  ‚   SINGLE LOW-9 QUOTATION MARK
U+2E32  ⸲   TURNED COMMA
U+2E34  ⸴   RAISED COMMA
U+2E41  ⹁   REVERSED COMMA
U+2E4C  ⹌   MEDIEVAL COMMA
U+3001  、   IDEOGRAPHIC COMMA
U+FE50  ﹐   SMALL COMMA
U+FE51  ﹑   SMALL IDEOGRAPHIC COMMA
U+FF0C  ，   FULLWIDTH COMMA
U+FF64  ､   HALFWIDTH IDEOGRAPHIC COMMA


### Colonlike Character Definition

A COLONLIKE CHARACTER is a colon (U+003A), or any other character in the following set: U+003A, U+02D0, U+02F8, U+0589, U+05C3, U+0703, U+0704, U+0903, U+0A83, U+0C03, U+0C83, U+0D03, U+16EC, U+205A, U+2236, U+2982, U+A789, U+FE13, U+FE30, U+FF1A

(Note: the only part of this that is currently theoretically important for parsing is `:` itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

This list includes things that could be visually mistaken for a colon in order to avoid confusion by the reader.

U+003A  :   COLON
U+02D0  ː   MODIFIER LETTER TRIANGULAR COLON
U+02F8  ˸   MODIFIER LETTER RAISED COLON
U+0589  ։   ARMENIAN FULL STOP
U+05C3  ׃   HEBREW PUNCTUATION SOF PASUQ
U+0703  ܃   SYRIAC SUPRALINEAR COLON
U+0704  ܄   SYRIAC SUBLINEAR COLON
U+0903  ः   DEVANAGARI SIGN VISARGA
U+0A83  ઃ   GUJARATI SIGN VISARGA
U+0C03  ః   TELUGU SIGN VISARGA
U+0C83  ಃ   KANNADA SIGN VISARGA
U+0D03  ഃ   MALAYALAM SIGN VISARGA
U+16EC  ᛬   RUNIC MULTIPLE PUNCTUATION
U+205A  ⁚   TWO DOT PUNCTUATION
U+2236  ∶   RATIO
U+2982  ⦂   Z NOTATION TYPE COLON
U+A789  ꞉   MODIFIER LETTER COLON
U+FE13  ︓   PRESENTATION FORM FOR VERTICAL COLON
U+FE30  ︰   PRESENTATION FORM FOR VERTICAL TWO DOT LEADER
U+FF1A  ：   FULLWIDTH COLON

### Underscorelike Character Definition

A UNDERSCORELIKE CHARACTER is a low line (U+005F), or any other character in the following set: U+005F, U+02CD, U+2581, U+23BC, U+23BD, U+FF3F

(Note: the only part of this that is currently theoretically important for parsing is `_` itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

The point of this is to avoid characters that look visually like an underscore to the point that the reader might mistake it for an underscore.

U+005F  _   LOW LINE
U+02CD  ˍ   MODIFIER LETTER LOW MACRON
U+23BC  ⎼   HORIZONTAL SCAN LINE-7
U+23BD  ⎽   HORIZONTAL SCAN LINE-9
U+2581  ▁   LOWER ONE EIGHTH BLOCK
U+FF3F  ＿  FULLWIDTH LOW LINE

### Foreslashlike Character Definition

A FORESLASHLIKE CHARACTER is a solidus (U+002F), or any other character in the following set: U+002F, U+1735, U+2044, U+2215, U+2571, U+29F8, U+FF0F

(Note: the only part of this that is currently theoretically important for parsing is `/` itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

The point of this is to avoid characters that look visually like a foreslash, to the point that the reader might mistake it for a foreslash.

U+002F  /   SOLIDUS
U+1735  ᜵   PHILIPPINE SINGLE PUNCTUATION
U+2044  ⁄   FRACTION SLASH
U+2215  ∕   DIVISION SLASH
U+2571  ╱   BOX DRAWINGS LIGHT DIAGONAL UPPER RIGHT TO LOWER LEFT
U+29F8  ⧸   BIG SOLIDUS
U+FF0F  ／  FULLWIDTH SOLIDUS

### Squarebracketlike Character Definition

A SQUAREBRACKETLIKE CHARACTER is a left square bracket (U+005B), a right square bracket (U+005D), or any other character in the following set: U+005B, U+005D, U+2772, U+2773, U+27E6, U+27E7, U+298B, U+298C, U+3010, U+3011, U+301A, U+301B, U+FF3B, U+FF3D

(Note: the only part of these that are currently theoretically important for parsing is [ and ] itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

The point of this is to avoid characters that look visually like a square bracket, to the point that the reader might mistake it for the `[ ` marker that marks an array level, the [] that spells an empty array, or the start or end of MINIMAL JSON.

This definition is not sided.  A definition that forbids this type of characters on both sides is forbidding both left and right both sides, not just when they are pointing in, or out, or in consistent or inconsistent directions.

U+005B  [   LEFT SQUARE BRACKET
U+005D  ]   RIGHT SQUARE BRACKET
U+2772  ❲   LIGHT LEFT TORTOISE SHELL BRACKET ORNAMENT
U+2773  ❳   LIGHT RIGHT TORTOISE SHELL BRACKET ORNAMENT
U+27E6  ⟦   MATHEMATICAL LEFT WHITE SQUARE BRACKET
U+27E7  ⟧   MATHEMATICAL RIGHT WHITE SQUARE BRACKET
U+298B  ⦋   LEFT SQUARE BRACKET WITH UNDERBAR
U+298C  ⦌   RIGHT SQUARE BRACKET WITH UNDERBAR
U+3010  【  LEFT BLACK LENTICULAR BRACKET
U+3011  】  RIGHT BLACK LENTICULAR BRACKET
U+301A  〚  LEFT WHITE SQUARE BRACKET
U+301B  〛  RIGHT WHITE SQUARE BRACKET
U+FF3B  ［  FULLWIDTH LEFT SQUARE BRACKET
U+FF3D  ］  FULLWIDTH RIGHT SQUARE BRACKET

### Curlybracketlike Character Definition

A CURLYBRACKETLIKE CHARACTER is a left curly bracket (U+007B), a right curly bracket (U+007D), or any other character in the following set: U+007B, U+007D, U+2774, U+2775, U+FF5B, U+FF5D

(Note: the only part of these that are currently theoretically important for parsing is { and } itself, but these avoid confusion, and preserve room for future expansion.)

If an implementor thinks something is visually confusing, they should absolutely use double quoting or other special treatment for greater clarity whether it's in this list or not.  This is just a limitation on what we parse.

The point of this is to avoid characters that look visually like a curly bracket, to the point that the reader might mistake it for the `{ ` marker that marks an object level, the {} that spells an empty object, or the start or end of MINIMAL JSON.

This definition is not sided.  A definition that forbids this type of characters on both sides is forbidding both left and right both sides, not just when they are pointing in, or out, or in consistent or inconsistent directions.

U+007B  {   LEFT CURLY BRACKET
U+007D  }   RIGHT CURLY BRACKET
U+2774  ❴   MEDIUM LEFT CURLY BRACKET ORNAMENT
U+2775  ❵   MEDIUM RIGHT CURLY BRACKET ORNAMENT
U+FF5B  ｛  FULLWIDTH LEFT CURLY BRACKET
U+FF5D  ｝  FULLWIDTH RIGHT CURLY BRACKET

---

## ADVANCED FEATURES

### MINIMAL JSON

MINIMAL JSON (JSON CONTAINING A NON-EMPTY OBJECT OR NON-EMPTY ARRAY WITH NO INTERMEDIATE WHITE SPACE):

This allows nonempty objects and non-empty arrays only as all simpler JSON values without extra whitespace are already also simple TJSON values already and require no additional restrictions or rules to support.  This section only applies to nonempty objects and arrays that the generator desires to present as MINIMAL JSON (strongly discouraged, but allowed).  MINIMAL JSON itself is a valid value within TJSON - MINIMAL means it contains no whitespace of any kind (aside from within double quoted strings of course in which case it is data), to include no real newline characters (real newline chars are not allowed in JSON anyway, so this is not an additional restriction).  Implementors should not actually generate MINIMAL JSON inside TJSON almost ever, but it is available as a last-ditch fallback if nothing else seems reasonable.

MINIMAL JSON MUST NEVER be wrapped or folded.  Wrapped or folded MINIMAL JSON is a parse error.

MINIMAL JSON MUST NEVER appear in a packed array format.  MINIMAL JSON in any packed array format is a parse error.

MINIMAL JSON also SHOULD NOT be packed by a GENERATOR (to include a re-formatter) in a TJSON line with any other value (not be packed in the sense of TJSON packing it with other items in the same object or array on the same line, it's required to be packed as far as no intermediate whitespace within the MINIMAL JSON itself) and can be detected with its opening `{[^} ]` or `[[^] ]` by the parser.  It ends at the first nondata whitespace character within the MINIMAL JSON, even if that makes it invalid.  If it isn't valid, that's a parse error.  All the basic JSON values work as is, and objects and arrays can be placed in there in MINIMAL JSON format if necessary - but this is almost never a good idea.  All the simple JSON values (not nonempty object, not nonempty array) are also TJSON values, so no special provision needs to be made for those.  A parser should allow MINIMAL JSON in packed kv pairs as described above and within tables (a directly analogous situation, MINIMAL JSON as a key in an object).  A parser should not allow MINIMAL JSON in packed array lines.  The reason that parsers must allow some of this and generators should not is to enable humans to hand edit without having to unpack lines or turn tables into a different format even though the result is ugly (but deterministically parsable!) and to enable extremely special case generators to generate it if it actually looks good or is otherwise a good idea for that use case.  This is almost always going to be a bad idea and a general case TJSON generator should not ever do this without some special option ordering it to do so.  The issues are that 1) it's hard to read and 2) it can look ambiguous.  Therefore, even if it parses under this specification, a generator should never output packed MINIMAL JSON without specific direction, either from a user option, or by the user choosing to use some extremely niche special case generator.  A packed array is easy for a human to hand split if they need to stuff some minimal json in on a line.  The human is guaranteed that if one half of the array is packable, the other one is too.  Packed arrays are type constrained by rule, as they are inherently more prone to ambiguity due to the smaller spacing involved, which would affect a human editor too.  Another reason for making an attempt to stick MINIMAL JSON in a packed array a parse error, is that it would otherwise look a bit like comma packed arrays.  We do not want to lose parse errors that might be highly informative to the hand editor in that context.

(Implementor's note: If you are highlighting or feeding MINIMAL JSON to an external JSON parser, it's easy to find a potential ending of valid minimal json via finding a whitespace character with 0 double quote parity while walking the string and not counting escaped \" while switching the parity bit if you are avoiding a full JSON parse, but this is solely an implementation note not a part of the spec and you can do it however you like as long as you do not turn a parse failure into a parse success.   We are erroring out completely rather than attempting to continue to parse if it fails so the exact manner in which you resolve this should not ever create a parse success where it otherwise would have failed.)

MINIMAL JSON is generally going to look better if it's broken out at an array boundary, because if it's broken out at an object boundary our only choice is to stuff it onto the same line as its parent key to avoid array ambiguity.

While producing TJSON containing MINIMAL JSON is strongly discouraged, it might be the least bad option for some implementations in some situations.  An example might be a single string within a 500 deep nested single element array or something similarly bizarre.  That's going to look like garbage no matter what you do, and TJSON containing MINIMAL JSON might be the best answer in some cases.

MINIMAL JSON SHOULD be on a line by itself (aside from the indent and the `[` or `{` indent marks that sometimes go within the indent), it must have zero non-data whitespace, (that's why it's called MINIMAL JSON) - and, except for kv/table packing in order to facilitate hand edits and niche generators described above, it always ends at the end of the line - nothing may come after it on that line and it must have the line to itself other than the indent and any indent marks before it.

As a special exception we allow a non folded bare or quoted key immediately before the MINIMAL JSON on its same line.  This is even more strongly discouraged than MINIMAL JSON, but the only way to do non-root level MINIMAL JSON that does not have an array level to break on.

MINIMAL JSON IS STRONGLY DISCOURAGED, BUT IS ALLOWED FOR USE WHEN NOTHING ELSE FITS YOUR USE CASE

**Example: TJSON containing only MINIMAL JSON (identity — the JSON output is exactly the same)**
```
{"a ":{"b":3,"c":"ab cd"}}
```
JSON version (intentionally exactly the same):
```json
{"a ":{"b":3,"c":"ab cd"}}
```

**Example: TJSON containing MINIMAL JSON (one `[]` layer is in the TJSON, shown by the n=2 indent level)**
```
  [{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]
```
JSON version (which also happens to be valid TJSON as it contains no extra whitespace and MINIMAL JSON is itself a TJSON value):
```json
[[{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]]
```

**Example: TJSON containing MINIMAL JSON (two `[]` array layers are in the TJSON, shown within the n=4 indent)**
```
[ [ [{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]
```
JSON version:
```json
[[[{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]]]
```

**Example: TJSON containing MINIMAL JSON (one object layer and one array layer is in the TJSON)**
```
  tjson_key:
    [{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]
```
JSON version:
```json
{"tjson_key":[[{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]]}
```

**Example — not recommended because MINIMAL JSON is not entirely on its own line, but we have to allow this as a special exception as otherwise it's not possible to embed MINIMAL JSON unless there's an array surrounding it to break at.**
```
  tjson_key:[{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]
```

**Example: TJSON containing not-quite minimal MINIMAL JSON**

*(Forbidden — not minimal: there is non-data whitespace in the JSON before null, and also forbidden independently because there is non-data whitespace after the comma near the end)*
```
  [{"a ":{"b": null,"c":"ab cd","d":3}}, {"a":true}]
```

**Bad Examples: TJSON containing MINIMAL JSON (one array layer is in the TJSON, with an additional true in the TJSON array)**

*(Forbidden — it's packed)*
```
  true  [{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]
```

*(Forbidden because it's packed, and independently forbidden because it's not at the end of the line)*
```
  [{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}], true
```

*(Forbidden — it's packed; though this is less bad as at least it's at the end of the line, it's still forbidden)*
```
  true, [{"a ":{"b":null,"c":"ab cd","d":3}},{"a":true}]
```

*(Forbidden — it's wrapped)*
```
  true
  [{"a ":{"b":null,"c":"ab cd","d":3}}
  / ,{"a":true}]
```


### Visible Indent Level Adjustment Glyphs

VISIBLE INDENT LEVEL ADJUSTMENT GLYPH: (it goes in the place where any value would go (cannot be used as a cell in a table obviously as it's not a BASIC TYPE), has leading space like a bare string, sets effective n (the visible nesting level) to 0, does not change actual nesting level n (this feature requires we keep track of a displayed indent offset or something like that in order to properly render))

THE VISIBLE INDENT LEVEL ADJUSTMENT GLYPH MUST BE IMMEDIATELY FOLLOWED BY AN EOL

THE PARSER SHOULDN'T REQUIRE THE OPENER TO BE THE ONLY VALUE ON ITS LINE - but generators should make it the only value on its line other than possibly keys and indent markers
```
 /<
```

VISIBLE INDENT LEVEL RESTORATION GLYPH: (MUST BE ALONE ON A LINE ASIDE FROM N REGULAR SPACE INDENTATION BEFORE, NO EXCEPTIONS) (there is a space before it just like a bare string, but bare strings are not allowed to start with `/`)
```
 />
```

Example at n=2 (probably unlikely unless a generator is trying to left justify a table, as 2 is too low to need INDENT LEVEL adjustment):
```
   />
```

Example at visible n=8 (if this is the first fold adjustment glyph, it's also going to be at logical n=8, but n will drop to 0 starting at the start of the next line after the fold adjustment glyph):
```
         />
```

or logically: `(n spaces) />EOL`

INDENT LEVEL ADJUSTMENT GLYPHS ARE NOT ALLOWED IN CANONICAL TJSON

AN ENDING INDENT LEVEL GLYPH IS ALLOWED BUT NOT REQUIRED IF YOU DON'T HAVE ANY MORE DATA BELOW THE INDENT LEVEL

**Example: deep object using indent level adjustment glyph** (this is only an example, don't do this unless indent level gets extreme; implementations do not have to do this at all; treating n=22 as a big number here; this can also be done around a table for stylistic reasons)

```json
{"a":{"b":{"c":{"d":{"e":{"f":{"g":{"h":{"i":{"k":{"l":{"m":{"n":{"o":{"p":{"q":{"r":{"s":{"t":{"u made longer for depth illustration note we restore to n not position of glyph":{"v":["about time","this is an array of strings","bare strings","and now I'm done"]},"sibling of u made longer for illustration bare key":"sibling of u bare string"},"sibling of t bare key":"sibling of t bare value"}}}}}}}}},"sibling of i bare key":"sibling of i bare string"},"sibling of h bare key":9}}}}}}}}}
```
```
  a:
    b:
      c:
        d:
          e:
            f:
              g:
                h:
                  i:
                    k: /<
// as an implementor I might choose to add a comment here sometimes because this indent change is confusing, or I might not to make it less obvious and allow
// the data to stand out rather than the data structure.  By default we do not add comments near INDENT LEVEL glyphs, though we do use them
// to offset big tables, but that's not really confusing like this 20 level object chain is.
  l:
    m:
      n:
        o:
          p:
            q:
              r:
                s:
                  t:
                    u made longer for depth illustration note we restore to n not position of glyph: /<
  v:
     about time
     this is an array of strings
     bare strings
     and now I'm done
                     />
                    sibling of u made longer for illustration bare key: sibling of u bare string
                  sibling of t bare key: sibling of t bare value
                     />
                    sibling of i bare key: sibling of i bare string
                  sibling of h bare key:9
                  // This is confusing and should be avoided if at all possible but may be the least bad option in some cases
                  // implementors may in some cases want to add comments to make it more obvious what is happening if they need to do this
```

**Example: offset table using indent level adjustment glyph** (a good idea when table width fits in w + 2 but not (w - n), and n is already a good chunk of w; n\*5>=w by default; not used by default if w is infinite)

Not coincidentally, when you print a table that's the root element it ends up looking exactly like this because the table is printed with the `|` at the object level n not the surrounding `[]` level n.  This is nice because it leaves room for a fold marker, so folding can be allowed.

```json
{"a":{"b":{"c":{"d":{"e":{"f":{"g":{"h key watch this":[{"look":1,"at":2,"my":true,"super":false,"cool":5,"table":"table is"},{"look":"too","at":"short","my":"though","super":"sadly","cool":"for"},{"look":"this","at":"technique","my":"to","super":"make","cool":"sense","table":"here"},{"look":"but","at":"for","my":"longer","super":"tables","cool":"it helps","table":"make it easier to read"},{"look":"Misalignment","at":"not","my":"great","super":"but","cool":"it is allowed"}],"sibling of h key watch this":"sibling of h value"}}}}}}}}
```
```
  a:
    b:
      c:
        d:
          e:
            f:
              g:
                h key watch this: /<
 // Optionally   /< here is where I would put the indent glyph instead if a generator wanted it on the next line, it would be the only two nonspace characters on the line if it weren't embedded in this comment, but it would be in exactly the same place
  |look  |at  |my  |super  |cool  |table  |
  |1     |2   |true  |false  |5   | table is  |
  | too  | short  | though  | sadly  | for  |
  | this  | technique  | to  | make  | sense  | here  |
  | but  | for  | longer  | tables  | it helps  | make it
  /  easier to read  |
  | Misalignment  | not  | great  | but  | it is allowed  |
                 />
                sibling of h key watch this: sibling of h value
```
**Example: above but generator pushing indent glyph to next line for stylistic reasons.**
```
  a:
    b:
      c:
        d:
          e:
            f:
              g:
                h key watch this:
                 /<
  |look  |at  |my  |super  |cool  |table  |
  |1     |2   |true  |false  |5   | table is  |
  | too  | short  | though  | sadly  | for  |
  | this  | technique  | to  | make  | sense  | here  |
  | but  | for  | longer  | tables  | it helps  | make it
  /  easier to read  |
  | Misalignment  | not  | great  | but  | it is allowed  |
                 />
                sibling of h key watch this: sibling of h value
```

THEORY BEHIND ABOVE EXAMPLE:
We reset n to 0, so now all we have to do is show the table JSON `[{....}]` at n=0, table gets shown starting at the indent level inside the surrounding array (the same level as the one at which we start the contained objects) not at the indent level before of the surrounding array, so we are effectively at n>=2.  Note that the table lines of the TJSON at EXACTLY the same visible indent as they would be if we just outputted the array of objects part of the above example as the root node of its own example.  This is not a coincidence, it must be this way to keep logical coherence.  It would be at the same visible indent level if it were the root node.  As you can see below, if you want a fully left justified table you can do it, by taking care of the array level indent first.

**Example: above but generator taking care of table array level indent before the indent adjustment glyph instead of after it to fully left justify table.**

```
  a:
    b:
      c:
        d:
          e:
            f:
              g:
                h key watch this:
                   /<
|look  |at  |my  |super  |cool  |table  |
|1     |2   |true  |false  |5   | table is  |
| too  | short  | though  | sadly  | for  |
| this  | technique  | to  | make  | sense  | here  |
| but  | for  | longer  | tables  | it helps  | make it
/  easier to read  |
| Misalignment  | not  | great  | but  | it is allowed  |
                   />
                sibling of h key watch this: sibling of h value
```

```json
{"barekey":3}
```
**Example with normal rendering**
```
  barekey:3
```
**Example with indent adjustment marker (probably a poor generator choice, but valid)**
```
   /<
barekey:3
   />
```
**Bad Example - an indent adjustment glyph must come at the start and end of a nonempty array or nonempty object - 3 is a basic value so this is invalid and should not parse**
```
  barekey: /<
3
```

```json
{"barekey":[3]}
```
**Example with indent adjustment marker (ugly but valid)**
```
  barekey: /<
  3
   />
```
**Example with indent adjustment marker (ugly but valid)**
```
  barekey:
   /<
  3
   />
```
**Bad Example: can't do inline array on a key with an indent adjustment glyph, must be on next line**
```
  barekey:   /<
3
```

```json
{"barekey":[[3]]}
```
**Example with normal rendering**
```
  barekey:
  [ [ 3
```
**Example with indent adjustment marker (ugly but valid)**
```
  barekey:
  [  /<
[ 3
     />
```
**Bad Example - indent markers must be provided if we do a jump of more than n+2 at once logically, and an indent adjustment glyph doesn't change any of that just because one of the n+2 jumps is on the next line.**
```
  barekey:
  [  /<
  3
     />
```
**Bad Example - indent markers must be provided if we do a jump of more than n+2 at once logically, and an indent adjustment glyph doesn't change any of that just because one of the n+2 jumps is on the next line.**
```
  barekey:
     /<
[ 3
     />
```


**Example: ending glyph optional when no more data follows** (allowed but not required to include closing `/>`)

```json
{"a":{"b":{"c":{"d":{"e":{"f":{"g":{"h":{"i":{"j":{"k":5}}}}}}}}}}}
```
```
  a:
    b:
      c:
        d:
          e:
            f:
              g:
                h:
                  i:
                    j: /<
// I am allowed to add an extra indent level ending glyph on the next line after the k:5, but I don't have to, because I don't have any more data that requires less logical indent to represent.
  k:5
// It would be here  />
```
```
  a:
    b:
      c:
        d:
          e:
            f:
              g:
                h:
                  i:
                    j:
                       /<
// This is the same idea but I took care of the indent BEFORE i used the indent adjustment glyph so I can
// now fully left justify k:5 rather than have two spaces before.  I still don't need an ending glyph.
k:5
// It would be here    />
```
**Example: deep array glyph use** (30 levels of array; we are allowed but not required to include the two closing `/>`, as there is no more data after either of the `/>` symbols - the placement of the indent glyphs is up to the generator, but as you can see we can't skip any indent markers just because we are using indent glyphs)
```json
[[[[[[[[[[[[[[[[[[[[[[[[[[[[[[0]]]]]]]]]]]]]]]]]]]]]]]]]]]]]]
```
```
[ [ [ [ [ [ [ [ [ [  /<
[ [ [ [ [ [ [ [ [ [  /<
[ [ [ [ [ [ [ [ [ [ 0
                     />
                     />
```

INFERRING INDENT MARKERS PAST DEFAULT BEHAVIOR (DON'T DO THIS!):

I'm not putting this in here so much because this is a good idea, but because invitably someone will THINK its a good idea (and it might be for certain rare hand editing cases with a local, non-default, and explicit parser option, even though it violates the spec) and I want them to do it in a way that will not break the core principles of TJSON.  If you are going to infer (FORBIDDEN!) indent markers beyond the extent in the specification where we normally skip them for one level steps, depth always wins and is never inferred under any circumstances, and we never fail to honor an indent marker at a certain line at a certain depth, or reinterpret an indent marker or anything else to be at a different depth or on a different line even if it's only off by one.

An array or object can never have more than one indent marker.  An indent marker starts an array or object (though it can be omitted if there's only one array or object starting on that line as it's obvious in that case only), and is often necessary to maintain parse certainty even when we aren't going up a level.  This is why the parser doesn't demand an indent marker for a single level, as it is obvious what is meant and it ends up being meaningless visual noise to most readers.

With just one nonempty array or object being created on that line, there's no uncertainty, which is why the default generator behavior is not to show an indent marker on those lines without some sort of force marker option being set.  This is part of the reason why inferring them beyond this is inherently hazardous and misleading to the reader.  Fewer wrapper objects+arrays is the prefered interpretation, but not if it violates any of the above.

If you are going to try to fill something in (not within this specification), DEPTH MUST ALWAYS WIN.  It doesn't matter what the indent markers say before it on a line, if its at depth n, its at depth n in the outgoing data, and it starts an object or array on that line.  No exceptions (reindent glyphs are the sole thing that can change this, but we are still honoring the depth over everything else even if it's modified by the indent marker context).  Reindent glyphs are a necessary evil to deal with finite rectangular screens, sadly.

Even following the above, actually implementing inference past what the default behavior does in a way that doesnt break everything else is theoretically possible but unreasonably difficult, and doing it wrong will cause collateral damage.  Also, it would likely surprise the user even if it were implemented perfectly.  Don't do it.

MOTIVATIONS AND TRADEOFFS REGARDING INDENT LEVEL ADJUSTMENT GLYPHS:

Indent level adjustment glyphs are in some ways contrary to the goals of the project.  It necessary to provide generators with a way to fit large and deep trees into finite display widths, and while we can adjust the format itself to stave off this unfortunate necessity as long as possible, there becomes a point at which an indent level adjustment might become superior overflowing and possibly forcing the user to side scroll.  Sadly, indent level adjustment glyphs in most contexts go against the principle that you should know where you are in the tree to some extent just by looking at the position of the indent on your page, as it changes the meaning of a visible indent.  However, in some contexts side scrolling is difficult or impossible, and on a printed page, automatic text wrapping destroys the integrity of the output.

In certain circumstances, such as tables, using indent adjustment glyphs by default before and after enhances readability by forcing the table to the left side of the reading area where the user might expect to find a table, creates room for additional data, and has only a small impact on positional awareness as long as we reliably adjust indent before and after the table.  This should feel familiar to a reader that also looks at ``` and left indented `` multiline strings, as it is more likely to feel like a display method for a field rather than an indent level, even if it is not necessarily that on a syntax or parsing level.

Adjusting the indent level for more than just the duration of a table or a multiline string in order to accommodate deeply indented trees is inherently more hazardous to the locality of the viewing experience, even though it may be necessary.  As such, generators may choose to, but need not, add comments near the indent level adjustment to orient the reader.  The form that these comments might take, or whether to generate them at all is left up to the generator, but I would suggest something like this to tell the user what visible indent level means if that is important to your application.  For many purposes, perhaps most, the reader may be less interested in the exact depth, and comments may be better omitted.  This commenting idea is not part of the rules of the specification, it is just a helpful note to remind generators that they can use comments to orient the reader, particularly when the generator is forced to do something disorienting to the reader.

Indent adjustment glyphs may only be used 1) before a nonempty array, ending immediately after the nonempty array ends, or 2) before a nonempty object starts, ending immediately after the nonempty object ends.

Comments must always be ignored by parsers.  The comments below are purely optional, and may, or may not be generated purely to orient the reader, in this case as to the actual n indent level of visible n=0.  Generators are allowed to add as many or as few comments as they like, to anything, for any reason, but hopefully with the goal of improving the reading experience.  As always, comments MUST be on their own line as required by the specification.

AN EXAMPLE OF USING COMMENTS TO ANCHOR AND DEFINE THE LEFT MARGIN AS A NONZERO INDENT LEVEL AND INFORM THE READER REGARDING THE ACTUAL INDENT LEVEL
```
  something:
    somethingelse:
     /<
//4
(more tjson, indent level is now n=4 even though visible n=0 after the indent glyph, the //4 at the very start of the line indicates that visible n=0 is really n=4)
//4
     /<
//8
(more tjson, indent level is now n=8 even though visible n=0 after the indent glyph, the //8 at the very start of the line indicates that visible n=0 is really n=8)
//8
     />
//4
    (more tjson, indent level is now n=8, visible n=4 after the indent glyph, the //4 at the very start of the line indicates that visible n=0 is really n=4)
//4
     />
//0
    (more tjson, indent level is now both visible n=4 and actual n=4, the //0 indicates that visible n=0 is really n=0)
```


---

## CANONICAL TJSON

CANONICAL TJSON can be thought of as a single generator option that produces a maximally consistent and mostly diff-friendlier alternative set of starting options for TJSON.  It has an infinite width and most features disabled (tables, multiline strings, indent level glyphs, line folding, etc.) for the sake of maximum uniformity at all costs.  It's also better at avoiding diff churn at the leaf value level.  This isn't going to be the right default for everyone, including those very interested in avoiding diff churn.  Tables and multiline strings can also be pretty good (in some cases arguably better) at minimizing diff churn depending on what changes you are likely to encounter.  Some users will want to start with canonical TJSON, and just allow one or two things, for instance multiline strings.  It's useful to have a more conservative default available to users that they can use a smaller number of options from rather than making them manually prohibit most things.  As such, if one option is canonical, and you use any other option that isn't already consistent with that, you are not outputting CANONICAL TJSON anymore, but it's a lot easier for the user to do that than specify each and every thing in CANONICAL TJSON except one, making a summary option like CANONICAL useful.

Here are the preferences for CANONICAL TJSON:

- Width is infinite (not the default)
- MINIMAL JSON is never used (the default)
- BARE KEYS (bare JSON object keys) are always used when allowed (the default)
- BARE STRINGS are always used when allowed (the default)
- MULTILINE STRINGS are not allowed (not the default)  (multiline requires at least one EOL inside the string, the default is to prefer MULTILINE strings for sufficiently long data that looks like text)
- double-quoted strings are the only allowed fallback for when BARE STRINGS are not allowed.
- MULTILINE arrays are preferred (no same line items even if they fit, not the default)
- MULTILINE objects are preferred (no same line keys even if they fit, not the default)
- AN OBJECT KEY MUST HAVE ITS VALUE ON THE SAME LINE IF IT'S A BASIC TYPE (boolean, null, number, string, or empty object/array/string)
  - This rule includes the starting glyph of a multiline string, which matters only if someone uses both canonical and allows multiline strings.  Normally multiline string glyphs can optionally start on the next line when multiline is enabled.
- AN OBJECT KEY MUST NOT have its value on the same line if its value is NOT a BASIC TYPE (not the default; this is canonical only because we disable packing)
- TABLES are forbidden (tables are great though, so implementations shouldn't default to CANONICAL TJSON if they don't need to have the highest possible line by line diff consistency, also not the default, but does have the secondary advantage of using slightly fewer cpu cycles)

Following the preferences above creates CANONICAL TJSON - actual implementations will often want to prefer one line arrays (partial, or perhaps for the whole array) and one line objects (partial, or for the whole object) where keys and values are short and there are not too many of them.  Canonical representation chooses multiline arrays and multiline objects because there is only one way to do a purely multiline array or object.

---

## SPECIFICATION CONSISTENCY

THIS IS THE SPECIFICATION: A future formal RFC type spec should be derived from this - it's supposed to be more parseable by LLMs and such - but if there is any difference in the rules whatsoever (and there shouldn't be), the RFC type document is the bug, not this.  Any underspecification here specified in a future RFC document is a bug there for specifying something there that's not defined or logically and necessarily implied here, and a bug here for not specifying it, as underspecification here is pointed out explicitly.
