# Self-documenting code and its limits — primary-source research

Scope: writing code that needs few comments, and where the recognized limits are (design rationale, why-not, workarounds, invariants, abstractions). Primary sources only: author's own book text, blog, or official docs. Book text was read from full-text PDFs; page refs are to the printed editions.

Sources ordered roughly chronologically.

---

## 1. Kernighan & Plauger, *The Elements of Programming Style* (1974; 2nd ed. 1978)

- URL: no free primary text online. Rules verified through direct citation in later primary sources: Clean Code ch. 4 epigraph cites "[KP78], p. 144"; McConnell ch. 32 cites "(1978)"; Henney (ACCU 2020) cites the "zero (or negative) value" line.
- Author: Brian W. Kernighan, P. J. Plauger. Date: 1974 / 1978.
- Claims (as quoted by the sources above):
  - "Don't comment bad code — rewrite it." (p. 144, quoted verbatim as Clean Code ch. 4 epigraph; McConnell renders it "Don't document bad code—rewrite it")
  - "A comment is of zero (or negative) value if it is wrong." (quoted by Henney)
  - "Make sure comments and code agree."
  - "Don't just echo the code with comments — make every comment count."
- Where it says code cannot carry the information: it does not; the book is the origin of the "rewrite, don't comment" line that every later source repeats.

---

## 2. Rob Pike, "Notes on Programming in C" (21 Feb 1989)

- URL: https://www.lysator.liu.se/c/pikestyle.html (mirror of Pike's Bell Labs memo; doc.cat-v.org copy is paywalled at time of fetch)
- Author: Rob Pike. Date: 1989-02-21.
- Claims:
  - "I tend to err on the side of eliminating comments, for several reasons."
  - "If the code is clear, and uses good type names and variable names, it should explain itself."
  - "Comments aren't checked by the compiler, so there is no guarantee they're right, especially after the code is modified."
  - "Comments clutter code."
  - Comments he keeps: introductions to global variables, "unusual or critical" procedures, marking sections of a large computation.
  - Ridicules `i=i+1; /* Add one to i */`.
  - "basically, avoid comments. If your code needs a comment to be understood, it would be better to rewrite it so it's easier to understand."
  - On names: "length is not a virtue in a name; clarity of expression is." Short `i` for loop index; longer names for rarely used globals (`maxphysaddr`).
- Where it says code cannot carry the information: only implicitly — he still comments globals and "unusual" procedures. Strongest pro-minimal-comment primary source.

---

## 3. Steve McConnell, *Code Complete*, 2nd ed. (2004), ch. 32 "Self-Documenting Code"

- URL (publisher page, paywalled): https://www.oreilly.com/library/view/code-complete-2nd/0735619670/ch32.html ; checklist mirror distributed under McConnell's cc2e.com permission: https://github.com/janosgyerik/software-construction-notes/blob/master/checklists-all/ch32-self-documenting-code.md . Full chapter text read from a PDF of the published 2nd edition (pp. 777–817).
- Author: Steve McConnell. Date: 2004.
- 32.3 "To Comment or Not to Comment" is staged as a Socratic dialogue. Key lines:
  - Anti-comment position given a fair hearing: "Comments are less precise than a programming language." "Refactoring eliminates most of my comments."
  - The rebuttal to "everything you need to know should be in the code": "Everything the compiler needs to know is in the code! You might as well argue that everything you need to know is in the binary executable file! ... What is meant to happen is not in the code."
  - "Good comments don't repeat the code or explain it. They clarify its intent. Comments should explain, at a higher level of abstraction than the code, what you're trying to do."
  - Comments as table of contents: "It's a lot faster to read one sentence in English than it is to parse 20 lines of code."
  - "If it's hard to comment, either it's bad code or you don't understand it well enough."
  - Verdict: "We'll encourage commenting, but we won't be naive about it."
- 32.4 "Keys to Effective Comments":
  - Three "mystery routines" show: wrong comment (Fibonacci labelled as sums), redundant comment (power routine), useful comment (Newton-Raphson). "Poor comments are worse than no comments."
  - **Kinds of Comments** (six in the published edition; the 2004 review draft lists five and lacks the sixth):
    1. **Repeat of the Code** — "restates what the code does in different words. It merely gives the reader of the code more to read without providing additional information."
    2. **Explanation of the Code** — for "complicated, tricky, or sensitive" code; "usually that's only because the code is confusing. If the code is so complicated that it needs to be explained, it's nearly always better to improve the code than it is to add comments."
    3. **Marker in the Code** — "isn't intended to be left in the code"; standardize on one token (e.g. TBD) so a release checklist can grep for it.
    4. **Summary of the Code** — "distills a few lines of code into one or two sentences"; scannable; useful when a non-author modifies the code.
    5. **Description of the Code's Intent** — "explains the purpose of a section of code. Intent comments operate more at the level of the problem than at the level of the solution." Example: `-- get current employee information` (intent) vs `-- update employeeRecord object` (summary). Cites IBM study (Fjelstad & Hamlen 1979): maintainers "most often said that understanding the original programmer's intent was the most difficult problem."
    6. **Information That Cannot Possibly Be Expressed by the Code Itself** — "Some information can't be expressed in code but must still be in the source code. This category of comments includes copyright notices, confidentiality notices, version numbers, and other housekeeping details; notes about the code's design; references to related requirements or architecture documentation; pointers to online references; optimization notes; comments required by editing tools such as Javadoc and Doxygen; and so on."
  - "The three kinds of comments that are acceptable for completed code are information that can't be expressed in code, intent comments, and summary comments."
  - Commenting Efficiently: "Too many comments are as bad as too few." If words don't come, "That's usually a sign that you don't understand what the program does." Use the Pseudocode Programming Process so "when you finish the code, the comments are done." "Integrate commenting into your development style" — commenting later "is also less accurate because you tend to forget assumptions or subtleties in the design."
  - Optimum number: Capers Jones/IBM: clarity peaked at "one comment roughly every ten statements"; but mandated ratios "address the symptom ... not the cause." "If the comments describe why the code was written ... you'll have enough comments."
- 32.5 "Commenting Techniques" (rules, bolded in the book):
  - "Focus paragraph comments on the why rather than the how" — how-comments "operate at the programming-language level rather than the problem level" and "are often redundant."
  - "Write comments at the level of the code's intent."
  - "Document surprises" — "If you find anything that isn't obvious from the code itself, put it into a comment"; for a performance trick "point out what the straightforward technique would be and quantify the performance gain."
  - "Justify violations of good programming style" — "That will prevent a well-intentioned programmer from changing the code to a better style, possibly breaking your code."
  - "Don't comment tricky code" — "Comments can't rescue difficult code. As Kernighan and Plauger emphasize, 'Don't document bad code—rewrite it.'" Cites Lind & Vairavan 1989: heavily commented regions had the most defects. "When someone says, 'This is really tricky code,' I hear them say, 'This is really bad code.'"
  - "Avoid endline comments on single lines."
  - Data: "Comment the units of numeric data"; ranges; coded meanings; "Document global data" — "the purpose of the data and why it needs to be global."
  - Routines: "Describe each routine in one or two sentences at the top" — "Difficulty in creating a short description is a sign that the design isn't as good as it should be." "Document interface assumptions" — legal/illegal values, sorted arrays, initialized member data.
  - Classes: "Describe the design approach to the class" — "Overview comments that provide information that can't readily be reverse engineered from coding details are especially useful. Describe the class's design philosophy, overall design approach, design alternatives that were considered and discarded."
- Where it says code cannot carry the information: kind 6 above ("Some information can't be expressed in code"), the class-level "design alternatives that were considered and discarded," and the dialogue line "What is meant to happen is not in the code."

---

## 4. Kernighan & Pike, *The Practice of Programming* (1999), §1.6 "Comments"

- URL: book site https://www.cs.princeton.edu/~bwk/tpop.webpage/ (no free excerpt of ch. 1; text read from a full-text PDF, pp. 23–27).
- Authors: Brian W. Kernighan, Rob Pike. Date: 1999.
- Claims:
  - "Comments are meant to help the reader of a program. They do not help by saying things the code already plainly says, or by contradicting the code, or by distracting the reader with elaborate typographical displays. The best comments aid the understanding of a program by briefly pointing out salient details or by providing a larger-scale view of the proceedings."
  - **Don't belabor the obvious.** `zerocount++; /* Increment zero entry counter */` — "All of these comments should be deleted; they're just clutter."
  - "Comments should add something that is not immediately evident from the code, or collect into one place information that is spread through the source."
  - **Comment functions and global data.** "A comment that introduces each function sets the stage for reading the code itself." For hard algorithms "a comment that points to a source of understanding can aid the reader. It may also be valuable to suggest why particular decisions were made." (IDCT example: cites the paper, describes data, states performance, "tells how and why the original algorithm has been modified.")
  - **Don't comment bad code, rewrite it.** "Comment anything unusual or potentially confusing, but when the comment outweighs the code, the code probably needs fixing." Example: rename `result` to `matchfound` and the comment disappears.
  - **Don't contradict the code.** "many a debugging session has been needlessly protracted because a mistaken comment was taken as truth."
  - "Comments should not only agree with code, they should support it." ctime newline example: rewrite to idiom `date[strlen(date)-1] = '\0'` and the comment "supports it by explaining why it needs to be there."
  - **Clarify, don't confuse.** `strcmp` header should just summarize behavior and cite "ANSI C, section 4.11.4.2".
  - "Comments are meant to help a reader understand parts of the program that are not readily understood from the code itself. As much as possible, write code that is easy to understand; the better you do this, the fewer comments you need. Good code needs fewer comments than bad code."
- Where it says code cannot carry the information: "collect into one place information that is spread through the source"; "suggest why particular decisions were made"; the reference/citation comment for the IDCT.

---

## 5. Martin Fowler, *Refactoring* (1999; 2nd ed. 2018), "Comments" smell; bliki "CodeAsDocumentation" (2005)

- URLs: 2nd-ed. smell excerpt https://www.informit.com/articles/article.aspx?p=2952392&seqNum=24 ; catalog https://refactoring.com/catalog/extractFunction.html , https://refactoring.com/catalog/extractVariable.html (alias "Introduce Explaining Variable"), https://refactoring.com/catalog/introduceAssertion.html ; bliki https://martinfowler.com/bliki/CodeAsDocumentation.html
- Author: Martin Fowler (Refactoring 1st ed. with Kent Beck et al.). Dates: 1999 / 2018; bliki 22 Mar 2005.
- "Comments" smell (ch. 3, last smell in the list):
  - "Don't worry, we aren't saying that people shouldn't write comments." The smell is that "comments are often used as a deodorant" — thick comments usually mask another smell.
  - Remedies: "If you need a comment to explain what a block of code does, try Extract Function. If the method is already extracted but you still need a comment to explain what it does, use Change Function Declaration to rename it. If you need to state some rules about the required state of the system, use Introduce Assertion."
  - Rule: "When you feel the need to write a comment, first try to refactor the code so that any comment becomes superfluous."
  - Legitimate comments: "A good time to use a comment is when you don't know what to do" (recording uncertainty), and "a comment is a good place to say why you did something" — "This kind of information helps future modifiers, especially forgetful ones."
  - Extract Variable (1st ed. name: Introduce Explaining Variable): replace a complex expression with named subexpressions (`basePrice`, `quantityDiscount`, `shipping`) so the name carries the explanation.
  - Introduce Assertion: `assert(this.discountRate >= 0)` — makes an implicit assumption explicit and executable, i.e. turns a would-be comment about an invariant into checked code.
- bliki "CodeAsDocumentation":
  - "code is the primary documentation of a software system" because "it is the only one that is sufficiently detailed and precise to act in that role."
  - "Code is no more inherently clear than any other form of documentation" — clarity requires deliberate effort and feedback ("there's nothing more important to clear code than getting feedback from others").
  - Explicitly not the *only* documentation: other documents still needed for what code cannot say.
- Where it says code cannot carry the information: the smell's exceptions — uncertainty ("don't know what to do") and "why you did something."

---

## 6. Jef Raskin, "Comments Are More Important Than Code" (ACM Queue, 18 Mar 2005)

- URL: https://queue.acm.org/detail.cfm?id=1053354
- Author: Jef Raskin. Date: 2005-03-18.
- Claims:
  - Code is not self-documenting and doc generators cannot fix that: "code can't explain why the program is being written, and the rationale for choosing this or that method."
  - Endorses Knuth's literate programming: "writing the documentation first, creating the methods in natural language, and describing the thinking behind them is a key to high-quality commercial programming."
  - "reconstructing code from good documentation is far easier than trying to create documentation given the code."
  - Against terse inline comments: forced brevity makes comments "laconic," hard to maintain, ultimately "useless"; prefers full-context block documentation.
  - His worked example (as relayed by Atwood 2006): a comment explaining why Boyer–Moore was chosen over binary search.
- Where it says code cannot carry the information: the "why ... and the rationale for choosing this or that method" line is the canonical statement.

---

## 7. Jeff Atwood, "Code Tells You How, Comments Tell You Why" (18 Dec 2006)

- URL: https://blog.codinghorror.com/code-tells-you-how-comments-tell-you-why/
- Author: Jeff Atwood. Date: 2006-12-18.
- Claims:
  - Quotes SICP: "Programs must be written for people to read, and only incidentally for machines to execute."
  - Quotes Raskin: "Code can't explain why the program is being written, and the rationale for choosing this or that method."
  - Refactoring example: replace magic numbers and cluttered inline comments with `DIRECTIONCODE_RIGHT`-style constants — naming removes the need for the comment.
  - Order of operations: "You should first strive to make your code as simple as possible to understand without relying on comments as a crutch. Only at the point where the code cannot be made easier to understand should you begin to add comments."
  - Title claim: code = how, comments = why. What is obvious to one developer is opaque to another without context.
- Where it says code cannot carry the information: rationale for method choice (via Raskin) — "only comments can explain" why one algorithm was rejected for another.

---

## 8. Jeff Atwood, "Coding Without Comments" (24 Jul 2008)

- URL: https://blog.codinghorror.com/coding-without-comments/
- Author: Jeff Atwood. Date: 2008-07-24.
- Claims:
  - Square-root example: a block with a comment "compute the square root using Newton-Raphson" becomes `SquareRootApproximation(n)` — "the code already tells us how it works; we need the comments to tell us why."
  - "write your code as if comments didn't exist" — comments only after refactoring for clarity.
  - Junior devs narrate code with comments; senior devs write code that needs little narration.
  - "If your feel your code is too complex to understand without comments, your code is probably just bad."
  - "to write good comments you have to be a good writer" — a separate skill most programmers lack.
- Where it says code cannot carry the information: "why" and design intent; everything else should be refactored into names.

---

## 9. Robert C. Martin, *Clean Code* (2008), ch. 4 "Comments"

- URL: publisher page https://www.oreilly.com/library/view/clean-code-a/9780136083238/chapter04.html (paywalled); text read from full-text PDF, pp. 53–74. Critique that quotes it at length: https://bugzmanov.github.io/cleancode-critique/chapter_4.html
- Author: Robert C. Martin. Date: 2008.
- Claims:
  - Epigraph: "Don't comment bad code—rewrite it." — Kernighan & Plauger.
  - "Nothing can be quite so helpful as a well-placed comment. Nothing can clutter up a module more than frivolous dogmatic comments. Nothing can be quite so damaging as an old crufty comment that propagates lies and misinformation."
  - "comments are, at best, a necessary evil. If our programming languages were expressive enough ... we would not need comments very much—perhaps not at all."
  - "The proper use of comments is to compensate for our failure to express ourself in code. Note that I used the word failure. I meant it. Comments are always failures."
  - "Every time you write a comment, you should grimace and feel the failure of your ability of expression."
  - "Why am I so down on comments? Because they lie. Not always, and not intentionally, but too often. The older a comment is, and the farther away it is from the code it describes, the more likely it is to be just plain wrong."
  - "Truth can only be found in one place: the code."
  - **Comments Do Not Make Up for Bad Code**: "Clear and expressive code with few comments is far superior to cluttered and complex code with lots of comments."
  - **Explain Yourself in Code**: "There are certainly times when code makes a poor vehicle for explanation. Unfortunately, many programmers have taken this to mean that code is seldom, if ever, a good means for explanation. This is patently false." `// Check to see if the employee is eligible for full benefits` → `if (employee.isEligibleForFullBenefits())`. "In many cases it's simply a matter of creating a function that says the same thing as the comment you want to write."
  - **Good Comments** (with the caveat "the only truly good comment is the comment you found a way not to write"): Legal Comments; Informative Comments (regex format — "better ... if this code had been moved to a special class"); **Explanation of Intent** ("provides the intent behind a decision"; `return 1; // we are greater because we are the right type.`; "This is our best attempt to get a race condition"); Clarification (translating obscure library return values — "there is no better way, and then take even more care that they are accurate"); **Warning of Consequences** (`//SimpleDateFormat is not thread safe, so we need to create each instance independently` — "will prevent some overly eager programmer from using a static initializer"); TODO Comments ("not an excuse to leave bad code in the system"); Amplification ("the trim is real important"); Javadocs in Public APIs.
  - **Bad Comments**: Mumbling; Redundant Comments (Tomcat javadocs "serve only to clutter and obscure"); Misleading Comments; Mandated Comments ("just plain silly to have a rule that says that every function must have a javadoc"); Journal Comments (source control does this); Noise Comments; Scary Noise; **Don't Use a Comment When You Can Use a Function or a Variable**; Position Markers; Closing Brace Comments; Attributions and Bylines; Commented-Out Code; HTML Comments; Nonlocal Information; Too Much Information; Inobvious Connection; Function Headers; Javadocs in Nonpublic Code.
- Where it says code cannot carry the information: "There are certainly times when code makes a poor vehicle for explanation"; the Good Comments list — intent behind a decision, warning of consequences, clarification of code "you cannot alter."

---

## 10. Kevlin Henney, "Comment Only What the Code Cannot Say" (97 Things, 2010; expanded ACCU Overload 157, Jun 2020)

- URLs: https://github.com/97-things/97-things-every-programmer-should-know/blob/master/en/thing_17/README.md ; https://accu.org/journals/overload/28/157/henney_2796/
- Author: Kevlin Henney. Dates: 2010; 2020-06.
- Claims:
  - "The difference between theory and practice is greater in practice than it is in theory" — in practice comments "often become a blight."
  - "a comment is of zero (or negative) value if it is wrong" (citing Kernighan & Plauger); wrong comments persist unlike wrong code, "a constant source of distraction and misinformation."
  - Noise sources: comments that restate code; commented-out code (goes stale, and the reason it was disabled is lost); version-history comments (VCS does it better).
  - Too many bad comments train readers to skip all comments, so good ones become invisible.
  - Prefer expressing things "in language structure, identifiers, and idioms" — rename rather than annotate a name, extract a function with an intent-revealing name rather than a section comment, rewrite bad code rather than apologize for it.
  - "Comments should say something code does not and cannot say."
  - "Comment what the code cannot say, not simply what it does not say." — the "does not say" gap should be closed by improving the code; only the "cannot say" remainder is legitimate comment territory.
  - "much of the skill in writing good comments is in knowing when not to write them."
- Where it says code cannot carry the information: the title thesis — intent, rationale, and non-obvious constraints that no identifier or structure can encode.

---

## 11. Peter Vogel, "No Comment" series, Visual Studio Magazine (2013)

- URLs: Part 1 "Why You Shouldn't Comment (or Document) Code" (27 Jun 2013) https://visualstudiomagazine.com/articles/2013/06/01/roc-rocks.aspx ; Part 2 "No Comment: Why Commenting Code Is Still a Bad Idea" (31 Jul 2013) https://visualstudiomagazine.com/articles/2013/07/26/why-commenting-code-is-still-bad.aspx ; Part 3 "Writing 'Really Obvious Code'" (12 Sep 2013) https://visualstudiomagazine.com/articles/2013/09/01/no-comment-part-3.aspx
- Author: Peter Vogel. Date: 2013.
- Claims:
  - "Comments that describe how the code achieves its goals are just a second version of the code" — you must read the code anyway to verify it, so the how-comment is redundant and rots.
  - Comments are "an expense" (written and maintained) and "the compiler doesn't check comments so there is no way to determine that comments are correct"; only code "communicates to the computer and should also communicate to whoever maintains the code."
  - Solution: "Really Obvious Code" (ROC) via refactoring under TDD. Part 3 rules: meaningful method names (`EnableDisableFormButtons`), rename controls (`DeleteOrdersButton` not `btDel`), extract complex conditions into named methods (`UnknownOrdersInCollection`), enums instead of magic strings, design patterns/classes for related behaviour (`UIManagementForManagers`), set defaults then override.
  - "The program's documentation (the ROC) is always both a complete and accurate description of what the program does."
  - Concessions ("three exceptions"): comment the *why* — business reason ("Normally, users can't delete orders"); "what the code should be doing" at method/class level (purpose, inputs/outputs, side effects); performance-forced non-obvious code, where the comment is "a kind of apology to the next programmer." "Commenting the 'why' and not the 'how' is a good first step."
  - Answers objections: readers who remember helpful comments "count the hits and forget the misses"; comments as training material are "really inefficient" and "counter-productive" given how often they are wrong.
  - Extends the claim to most technical documentation: ignored because developers know it is unreliable.
- Where it says code cannot carry the information: the why / business rule; the "apology" for performance hacks.

---

## 12. Linux kernel coding style, §8 "Commenting" (official docs)

- URL: https://www.kernel.org/doc/html/latest/process/coding-style.html#commenting
- Author: Linus Torvalds et al. Date: ongoing (text stable since the 1990s).
- Claims:
  - "Comments are good, but there is also a danger of over-commenting. NEVER try to explain HOW your code works in a comment: it's much better to write the code so that the working is obvious, and it's a waste of time to explain badly written code."
  - "Generally, you want your comments to tell WHAT your code does, not HOW."
  - "try to avoid putting comments inside a function body: if the function is so complex that you need to separately comment parts of it, you should probably go back to chapter 6 [Functions] for a while."
  - "You can make small comments to note or warn about something particularly clever (or ugly), but try to avoid excess."
  - "put the comments at the head of the function, telling people what it does, and possibly WHY it does it."
- Where it says code cannot carry the information: the function-header "WHY" and warnings about "something particularly clever (or ugly)."

---

## 13. Google C++ Style Guide, "Comments" (official docs)

- URL: https://google.github.io/styleguide/cppguide.html#Comments
- Author: Google. Date: maintained continuously.
- Claims:
  - "Comments are absolutely vital to keeping our code readable ... But remember: while comments are very important, the best code is self-documenting. Giving sensible names to types and variables is much better than using obscure names that you must then explain through comments."
  - "write for your audience: the next contributor who will need to understand your code. Be generous — the next one may be you!"
  - Declaration comments "describe use of the function"; definition comments "describe operation." Declaration comments "do not describe how the function performs its task."
  - Things to state at a declaration: inputs/outputs; whether the object keeps references beyond the call; nullability of pointer args and what happens; whether output args are appended to or overwritten; performance implications.
  - Definition comments: "any coding tricks you use, give an overview of the steps you go through, or explain why you chose to implement the function in the way you did rather than using a viable alternative" — e.g. why a lock is needed in the first half but not the second.
  - Class data members: "If there are any invariants (special values, relationships between members, lifetime requirements) not clearly expressed by the type and name, they must be commented. However, if the type and name suffice (`int num_events_;`), no comment is needed." Document sentinel values (`-1 means that we don't yet know...`).
  - Globals: what they are, what for, "and (if unclear) why they need to be global."
  - Function-argument comments are a "last resort" after named constants, enums instead of bools, options structs, and named variables.
  - "Do not state the obvious. In particular, don't literally describe what code does, unless the behavior is nonobvious ... Instead, provide higher-level comments that describe why the code does what it does, or make the code self-describing." Example progression: `// Find the element in the vector.` (bad) → `// Process "element" unless it was already processed.` → `if (!IsAlreadyProcessed(element))` ("Self-describing code doesn't need a comment").
- Where it says code cannot carry the information: invariants "not clearly expressed by the type and name"; why-this-implementation-not-the-alternative; ownership/lifetime of pointer args.

---

## 14. Salvatore Sanfilippo (antirez), "Writing system software: code comments" (2018)

- URL: http://antirez.com/news/124
- Author: Salvatore Sanfilippo. Date: 2018 (≈ Oct 2018; page shows relative date).
- Claims:
  - Rejects "if the code is solid it needs no comments": (a) many comments explain *why*, "why it's doing something that is clear instead of something else that would feel more natural" — information absent from the code; (b) comments are "a tool for lowering the cognitive load of the reader."
  - "a key goal in writing readable code is to lower the amount of effort and the number of details the reader should take into her or his head while reading some code."
  - **Nine comment types** (from a classification of Redis source):
    1. **Function comments** — API doc at the function definition; "the reader does not need to read the implementation" to use it. Good.
    2. **Design comments** — at the top of a file; the algorithms, techniques, and "why" behind the implementation, a high-level map before the code. Good.
    3. **Why comments** — explain why the code does something, especially when it does the non-obvious thing; capture reasoning the code cannot. Good; the most important type.
    4. **Teacher comments** — teach domain knowledge (math, networking, a protocol) the reader may lack; do not explain the code itself. Good.
    5. **Checklist comments** — "if you modify this, also change X, Y, Z"; reminders of coupled changes elsewhere. Good, when a better structure is impractical.
    6. **Guide comments** — divide code into sections and narrate flow; "sole reason to exist is to lower the cognitive load of the programmer reading some code"; give rhythm. Good (he defends these against the "obvious comment" charge).
    7. **Trivial comments** — "cognitive load of reading the comment is the same or higher than just reading the associated code." Bad.
    8. **Debt comments** — TODO/FIXME/XXX; technical-debt markers. Tolerable if tracked; questionable.
    9. **Backup comments** — commented-out old code; obsolete given version control. Bad.
  - "Writing good comments is harder than writing good code" — requires understanding the design and writing skill; but it is a skill that can be developed.
- Where it says code cannot carry the information: Why comments ("why it's doing something that is clear instead of something else"), Design comments (algorithm choice), Checklist comments (cross-file coupling), Teacher comments (domain background).

---

## 15. John Ousterhout, *A Philosophy of Software Design* (2018; 2nd ed. 2021), ch. 12–16; CS190 lecture notes

- URLs: lecture notes https://web.stanford.edu/~ouster/cgi-bin/cs190-winter18/lecture.php?topic=comments ; book text read from full-text PDF (ch. 12 pp. 101–106, ch. 13 pp. 107–128, ch. 14 pp. 129–136, ch. 15 pp. 137–141, ch. 16 §16.3).
- Author: John Ousterhout. Date: 2018 / 2021.
- **Ch. 12 "Why Write Comments? The Four Excuses"**:
  - Framing: "Documentation also plays an important role in abstraction; without comments, you can't hide complexity. Finally, the process of writing comments, if done correctly, will actually improve a system's design."
  - Excuse 1, **"Good code is self-documenting."** — "This is a delicious myth, like a rumor that ice cream is good for your health: we'd really like to believe it! Unfortunately, it's simply not true." Concedes naming reduces need, but "there is still a significant amount of design information that can't be represented in code. For example, only a small part of a class's interface, such as the signatures of its methods, can be specified formally in the code. The informal aspects of an interface, such as a high-level description of what each method does or the meaning of its result, can only be described in comments. There are many other examples of things that can't be described in the code, such as the rationale for a particular design decision, or the conditions under which it makes sense to call a particular method."
  - Rebuts "just read the method's code": it is "time-consuming and painful," and designing for it pushes you to "a large number of shallow methods," which "doesn't really make the code easier to read" because you must then read the nested methods too. "For large systems it isn't practical for users to read the code to learn the behavior."
  - Abstraction argument: "If users must read the code of a method in order to use it, then there is no abstraction: all of the complexity of the method is exposed." "Without comments, the only abstraction of a method is its declaration ... The declaration is missing too much essential information." substring(start, end) example: is `end` inclusive; what if start > end. "If you want to use abstractions to hide complexity, comments are essential." English is "less precise than code, but it provides more expressive power."
  - Excuse 2, "I don't have time" — comments add at most ~10% of dev time; interface comments "pay for themselves immediately" as design tools.
  - Excuse 3, "Comments get out of date" — "need not be a major problem"; keep docs near code, avoid duplication (ch. 16); "Code reviews provide a great mechanism for detecting and fixing stale comments."
  - Excuse 4, "All the comments I have seen are worthless" — "probably the one with the most merit"; fix by learning to write them.
  - §12.5: "The overall idea behind comments is to capture information that was in the mind of the designer but couldn't be represented in the code." Ranges "from low-level details, such as a hardware quirk that motivates a particularly tricky piece of code, up to high-level concepts such as the rationale for a class." Without them "future developers will have to rederive or guess at the developer's original knowledge." Comments reduce cognitive load and unknown unknowns; "Good documentation can clarify dependencies, and it fills in gaps to eliminate obscurity."
- **Ch. 13 "Comments Should Describe Things that Aren't Obvious from the Code"**:
  - "statements in a programming language can't capture all of the important information that was in the mind of the developer when the code was written." Guiding principle: "comments should describe things that aren't obvious from the code."
  - Non-obvious things: inclusive/exclusive ranges; "why code is needed, or why it was implemented in a particular way"; rules like "always invoke a before b" — "You might be able to guess at a rule by looking at all of the code, but this is painful and error-prone; a comment can make the rule explicit."
  - "Developers should be able to understand the abstraction provided by a module without reading any code other than its externally visible declarations. The only way to do this is by supplementing the declarations with comments."
  - §13.1 categories: Interface, Data structure member, Implementation, Cross-module. "Every class should have an interface comment, every class variable should have a comment, and every method should have an interface comment ... it is easier to comment everything rather than spend energy worrying about whether a comment is needed. Implementation comments are often unnecessary."
  - §13.2 Don't repeat the code. Test: "could someone who has never seen the code write the comment just by looking at the code next to the comment? If the answer is yes ... the comment doesn't make the code any easier to understand." Red Flag: Comment Repeats Code. "use different words in the comment from those in the name of the entity being described."
  - §13.3–13.4: "Comments augment the code by providing information at a different level of detail." Lower-level = **precision** (units; inclusive/exclusive boundaries; meaning of null; who frees the resource; invariants "such as 'this list always contains at least one entry'"). Higher-level = **intuition** ("the reasoning behind the code, or a simpler and more abstract way of thinking about the code"). "Comments at the same level as the code are likely to repeat the code." Variables: "think nouns, not verbs." Higher-level comments: "What is this code trying to do? What is the simplest thing you can say that explains everything in the code?" "How we get here" comments are valuable.
  - §13.5 Interface documentation: "Code isn't suitable for describing abstractions; it's too low level and it includes implementation details that shouldn't be visible in the abstraction. The only way to describe an abstraction is with comments." Interface comments must cover behavior, each argument/return, side effects, exceptions, preconditions. "If interface comments must also describe the implementation, then the class or method is shallow." Red Flag: Implementation Documentation Contaminates Interface.
  - §13.6 "Implementation comments: what and why, not how." "Most methods are so short and simple that they don't need any implementation comments." Block comments describe *what* at a higher level (`// Phase 1: Scan active RPCs...`); also *why* — "if a bug fix requires the addition of code whose purpose isn't totally obvious, add a comment describing why the code is needed" (e.g. "Fixes RAM-436").
  - §13.7 Cross-module design decisions: Status-enum example with a 7-step "if you add a new status value you must..." checklist; `designNotes` file with `// See "Zombies" in designNotes.` pointers.
  - §13.8: "there is a significant amount of information that can't easily be deduced from the code. Comments fill in this information." "'obvious' is from the perspective of someone reading your code for the first time (not you) ... if a reader thinks it's not obvious, then it's not obvious."
- **Ch. 14 "Choosing Names"**: "Good names are a form of documentation ... They reduce the need for other documentation." Sprite `block` bug (physical vs logical block) took six months. "Create an image"; names must be precise (`getCount` → `numIndexlets`; `blinkStatus` → `cursorVisible`; `x`,`y` → `charIndex`,`lineIndex`) and consistent. Red Flags: Vague Name; **Hard to Pick Name** ("a hint that the underlying object may not have a clean design"). §14.5 disagrees with Go's short-name culture (Gerrand's "long names obscure what the code does").
- **Ch. 15 "Write the Comments First"**: "Delayed comments are bad comments" — written after the fact "the comments repeat the code" and forget design ideas. Process: class interface comment → method interface comments and signatures with empty bodies → instance-variable comments → bodies. "Comments provide the only way to fully capture abstractions." "Comments serve as a canary in the coal mine of complexity. If a method or variable requires a long comment, it is a red flag that you don't have a good abstraction." Red Flag: Hard to Describe. Cost: comments ≤ ~5% of dev time; writing them first may be faster overall.
- **§16.3 "Comments belong in the code, not the commit log"**: a commit message "describing a subtle problem that motivated a code change" must also be in the code, else "a developer might come along later and undo the change without realizing that they have re-created a bug."
- Lecture notes (CS190) compress this to: "Comments should describe things that are not obvious from the code." Mistake #1: comments duplicate code. Mistake #2: non-obvious information isn't described. Write interface docs before method bodies as a design tool.
- Where it says code cannot carry the information: the whole of §12.1 and §12.5 — "design information that can't be represented in code": informal interface semantics, "the rationale for a particular design decision," "the conditions under which it makes sense to call" something, hardware quirks, cross-module rules, invariants. Plus §13.5: "The only way to describe an abstraction is with comments."

---

## 16. Hillel Wayne, "Comment the Why *and* the What" (14 Jun 2021) and "Why Not Comments" (10 Sep 2024)

- URLs: https://buttondown.com/hillelwayne/archive/comment-the-why-and-the-what/ ; https://buttondown.com/hillelwayne/archive/why-not-comments/
- Author: Hillel Wayne. Dates: 2021-06-14; 2024-09-10.
- "Comment the Why and the What":
  - Targets the slogan "comment the why, not the what," which rests on "code should be self-documenting." He disagrees: returning to old projects, comments are "far more useful to reorient" him than code or tests.
  - Self-documenting code "only reveals local structure"; readers cannot tell "why all of the extra stuff is there" (validation, auditing, logging, config) without knowing the core operation.
  - Five cases for *what* comments: (1) context — FitNesse's Shutdown class: the actual HTTP call `'/?responder=shutdown'` is spread across four methods in three files, one comment would show the path; (2) describing the essential algorithm buried under supporting code; (3) optimized code — "Performant code is going to be less clear than non-performant code"; (4) skill differentials (juniors, unfamiliar libraries); (5) language weirdness (bash array copy, VimScript operators).
  - Also: ASCII diagrams for 2-D information; developer-to-developer notes (warnings, speculation, negative information).
- "Why Not Comments":
  - Identifiers and tests can describe what code does; "negative information" — what the code deliberately does *not* do — cannot be embedded in code.
  - Distinguishes "why" (design decision) from "why not" (rejected alternative). A why-not comment shows a suboptimal choice "was a conscious choice, not an oversight."
  - Example from his epub_math_fixer.py: "Does 16 passes over each string BUT there are only 25 math strings in the book so far and most are <5 characters. So it's still fast enough."
  - Code cannot encode: tradeoffs and alternatives considered; counterfactuals; two independent clauses at once (purpose and design reasoning).
- Where it says code cannot carry the information: "negative information"/"why nots" — "'Self-documentation' describes what code does" while what it does not do, and why, has nowhere to live but a comment.

---

## Points of agreement across sources

1. **Refactor before commenting.** Every source, including the pro-comment ones, says a comment that explains *how* a block works is a signal to extract/rename (Kernighan & Plauger 1974 → K&P 1999 → Fowler → Atwood → Martin → Henney → Vogel → Google → kernel). Ousterhout agrees for implementation comments ("what and why, not how"; most methods need none).
2. **Repeating the code is the canonical bad comment.** `i++ /* increment i */` appears in Pike 1989, K&P 1999; McConnell "Repeat of the Code"; Martin "Redundant/Noise"; antirez "Trivial"; Ousterhout "Red Flag: Comment Repeats Code"; Google "Do not state the obvious."
3. **Wrong comments are worse than none.** K&P "zero (or negative) value"; McConnell "Poor comments are worse than no comments"; Martin "Inaccurate comments are far worse than no comments at all"; Henney; Vogel; Pike ("no guarantee they're right").
4. **Names do most of the work.** Pike, K&P (`matchfound`), Fowler (Extract Variable / rename), Martin (`isEligibleForFullBenefits`), Vogel (ROC rules), Google ("sensible names ... much better"), Ousterhout ch. 14 ("Good names are a form of documentation").
5. **"Why" is what remains after the code is clear.** Raskin, Atwood, Vogel, Fowler ("why you did something"), McConnell ("Focus ... on the why rather than the how"), Martin ("Explanation of Intent", "Warning of Consequences"), antirez ("Why comments"), Google ("why you chose to implement the function in the way you did rather than using a viable alternative"), kernel ("possibly WHY"), Ousterhout, Henney, Wayne.
6. **Specific things code cannot say (union across sources):** design rationale and rejected alternatives (Raskin, McConnell class docs, Ousterhout, Wayne "why not"); reasons for workarounds/style violations/performance hacks (McConnell "Justify violations", "Document surprises"; Vogel "apology"; Martin "Warning of Consequences"; Google lock example); invariants, units, inclusive/exclusive ranges, null meaning, ownership (Ousterhout §13.3; Google data members; McConnell data guidelines; Fowler → Introduce Assertion where checkable); cross-module coupling / "if you change this also change that" (Ousterhout §13.7; antirez "Checklist"; K&P "collect into one place information that is spread"); references to papers, bug IDs, specs (K&P IDCT; Ousterhout "Fixes RAM-436"; McConnell kind 6); legal/housekeeping (McConnell kind 6; Martin "Legal"; Google file comments); domain background (antirez "Teacher").
7. **Write comments as part of design, not afterwards.** McConnell (Pseudocode Programming Process, "commenting done later ... less accurate because you tend to forget assumptions"), Raskin/Knuth (documentation first), Ousterhout ch. 15 ("Delayed comments are bad comments"). And difficulty writing the comment is a design smell: McConnell ("Difficulty in creating a short description is a sign that the design isn't as good"), Ousterhout ("canary in the coal mine", Red Flag: Hard to Describe, Hard to Pick Name).
8. **Comments in commit logs are not enough.** Ousterhout §16.3 explicit; Martin and Henney say VCS replaces *journal* comments but not rationale.

## Points of disagreement

1. **Default stance: comment everything at the interface vs. comment as little as possible.**
   - Ousterhout: "Every class should have an interface comment, every class variable should have a comment, and every method should have an interface comment ... it is easier to comment everything." Google: "Almost every function declaration should have comments."
   - Martin: "Mandated Comments" is a *bad* comment category — "just plain silly to have a rule that says that every function must have a javadoc." Pike: "basically, avoid comments." Vogel: comments are "an expense."
2. **Are comments a failure?**
   - Martin: "Comments are always failures ... you should grimace and feel the failure of your ability of expression." Atwood, Vogel, Pike in the same spirit.
   - Ousterhout directly opposes: "Good code is self-documenting" is "a delicious myth"; comments are "essential" to abstraction and "will actually improve a system's design." antirez: "Writing good comments is harder than writing good code." Raskin: "Comments are more important than code." McConnell's dialogue gives the verdict to the pro-comment side.
3. **Can you read the implementation instead of the doc?**
   - Martin: "Truth can only be found in one place: the code."
   - Ousterhout: making users read method bodies means "there is no abstraction"; it also pushes toward "a large number of shallow methods" — an implicit critique of Clean Code's small-function style (the bugzmanov critique makes this explicit using Martin's own PrimeGenerator example).
4. **"What" comments.**
   - Atwood/Vogel: code = how/what, comments = why (Vogel says only why). The Linux kernel guide is on the other side here: "you want your comments to tell WHAT your code does, not HOW", at the function head.
   - Hillel Wayne: "comment the why *and* the what" — local structure does not reveal the essential algorithm under overhead, cross-file call paths, or language quirks. Ousterhout §13.6 agrees that implementation comments should say *what* at a higher level ("Phase 1: Scan active RPCs"). McConnell's "summary" comments are also what-comments and he accepts them.
5. **Extract-a-function vs. guide comments.**
   - Fowler/Martin: a section comment means Extract Function.
   - antirez: "guide comments" that section code are good because they lower cognitive load; Ousterhout: over-extraction yields shallow methods; McConnell's dialogue: "20 or 30 routine calls without needing any comments" is the skeptic's line, not the conclusion.
6. **Staleness risk.**
   - Martin/Pike/Vogel treat rot as near-inevitable ("Programmers can't realistically maintain them"; compiler can't check).
   - Ousterhout: "need not be a major problem in practice" — reviews, proximity, no duplication; McConnell: "finding a disagreement between the comment and the code tends to mean that both are wrong."
7. **Comment density.**
   - McConnell cites IBM data (peak clarity ≈ 1 comment / 10 statements) but rejects mandated ratios.
   - Martin/Pike give no target and treat any count as something to minimize; Ousterhout gives no ratio but a coverage rule (every interface, every member).
8. **Name length.**
   - Pike/Go: short names, "length is not a virtue."
   - Ousterhout §14.5 explicitly disagrees with Gerrand's Go slides; McConnell and Martin favour descriptive names.

## Access notes

- Read directly from the author's own site: Pike memo (mirror), Atwood ×2, antirez, Henney (GitHub + ACCU), Fowler (InformIT excerpt + refactoring.com + bliki), Ousterhout lecture notes, Hillel Wayne ×2, Linux kernel docs, Google style guide, Raskin (via text proxy), Vogel ×3 (via text proxy; site blocks direct fetch).
- Read from full-text PDFs of the published books: Code Complete 2e ch. 32 (published edition; a 2004 review-draft copy lists only five comment kinds — the sixth exists only in the published text), Clean Code ch. 4, The Practice of Programming §1.6, A Philosophy of Software Design ch. 12–16.
- Not obtained as primary text: Elements of Programming Style itself (rules confirmed via verbatim citations in Clean Code, Code Complete, Henney).
- Rejected as non-primary: Medium/dev.to summaries, LinkedIn summaries, goodreads notes, refactoring.guru, lecture slides summarizing McConnell (used only to locate passages, then replaced with book text).
