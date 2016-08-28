# System Design and Implementation {#Chapter4}

> *"Out of all the documentation that software projects normally generate, was
> there anything that could truly be considered an eningeering document?"* The
> answer that came to me was, *"Yes, there was such a document, and only one--the source code."*

**J. Reeves, 2001**

This chapter describes the most interesting aspects of the delivered system
architecture and implementation. First, the design principles followed during
development are described, then a high level overview is provided before moving
on to the

As mentioned before, the complete application is the product of a team effort,
with Deborah Wacks working on the web server implementation and the UX design,
Rim Ahsaini working on the specialized search engine, and the author working on the
chatbot brain. The scope of the author work is limited to the implementation of
the chatbot and supporting systems. The overall architecture of the components is illustrated

(#ISIT??)

## Software Architecture

### Principles of Software Design

SOLID principles of software design were used by the author while working on the source code;
these are dependency management principles (policing the "import" statements
throughout the project) and clean design principles, chief among them the Separation
of Concerns principle (Martin, 2003, Section 2).
Additionally, the code design guidelines advocated by Martin (2009) were discovered
by the author during the writing, and an attempt was made to apply them (in line with author's stated aims).
The chief purpose of these efforts is to design the system for change, and
make the code readable to *humans* (Martin, ibid, pp.13-14).
This approach leads to extremely specialized and short class bodies and functions, and
the separation of different levels of abstraction.
Additionally, the project was structured with the "plugin model" architecture:
ensuring that the direction of dependency flow in the direction of the core of the
system[^arguing].

The core of a chatbot system are the high level abstractions precisely defined in
the interfaces and the abstract classes (Dependency Inversion Principle).
The boundaries between systems insist that change in the external systems do not
affect the well functioning of the core of the system: this should not have
dependencies flowing outwards, with outward facing boundaries that expose interfaces
for the external systems to implement or use.

[^arguing]: These approaches is not free of controversy, but proceed from a very
influential and successful school of thought, hence the author is adopting it
in an effort to experiment and improve as a software engineer.

### High Level Structure

The folder structure of the chatbot is meant to be self-describing, the reader
should be able to understand what the purpose of the system and of the
sub-folders within without further assistance (see Martin, 2011).

Within the botinterface subdirectory we can find the high level
abstractions of the core system: the bot_abstract module defines the interface
to the bot application as a whole, with a very restrictive set of methods that
essentially make the abstraction a simple "response machine". Given a message,
an appropriate natural language reply is expected[^interfaces].

The brain folder contains the RiveScript language files that define the matchable
patterns and reply templates, the Python macros callable from within the brain,
and the topics structure as seen from the brain, that limits the scope of pattern
matchers depending on the the stage of the conversation between user and system.

The messagelog package defines a data model that may be useful to gather the data
about a conversation a user had with the system and extract the information to persist
for the purpose of the care plan creation. Again, the scope of the project is limited
to the architecture of the system, and while the larger PEACH team produced a relational
database schema to persist the data, it is not within the objectives of the author's
project to follow through with serializing the information to durable storage.
The package is there to be used in the future.

The concerns package encodes the information about a user's concerns, as well as
the range of acceptable macro and micro topics (or categories) for each concern.
The point of access to this subsystem is defined in the drive_conversation_abstract
module, as another interface. This interface describes the API to the part of
the module that keeps track of what the state user concerns are, and their priority
order. The concerns_factory module defines a class with static (or class-level)
methods for the purpose of keeping track of various user sessions in the system,
each represented by a DistressConversationDriver object (a concrete implementation
  of the ConversationDriver interface).

The categorize package exposes a set of modules to train, store to the file system
and retrieve evaluation metrics for single label classifiers. This package is
the least well designed element of the system primarily because it was the first
to be developed. The reason this subpackage was not used in the final pipeline
implementation has to do with the fact that it took too long for the survey
that was created to collect relevant data to produce results. But it is possible
in principle to plug a trained categorizer at the preprocessor level of the
message pipeline in order to add a tag to the user message, in order for this
to be used by the chatbot framework to provide better replies to the user.

Finally, the synonym package defines a SynonymExtractor interface and one implementation
for it. As noted in Chapter 2, this implementation did not perform to expectations,
and was therefore not used in the current system. However, the synonym package
should be easy to extent to allow (for example) a WordNet synonym generator
implementation, and the package already provides logic to represent the RiveScript
syntax data structure (the RiveScript "array") and a way to write this structure to file[^streamingRiveScript].

[^streamingRiveScript]: Additionally, see "Dynamically Streaming Data into a RiveScript Interpreter"
in this report for instructions on how to stream RiveScript syntax into a running
interpreter.

### The BotInterface, MessagePreprocessor and MessagePostprocessor layers

Looking at the one concrete subclass of this interface, we find it is in fact a
FAÇADE and a PROXY: it essentially delegates the processing of the natural language message
to other components of the system (Gamma et al, 1995, pp.185-193, 207-217; Martin, 2003, pp.327--). The modularity
of the design makes it easy to change implementation of these components, and provides
a clear and sensible separation of concerns with the message coming into the system
being preprocessed prior to being forwarded to the chatbot framework, and then
postprocessed as needed. This provides a degree of decoupling from the chatbot
framework, instead of making it a central component of the system, allowing it to be changed for
another one of the options surveyed in Chapter 2 with relative ease (in this sense it is a PROXY:
  it "looks" like the underlying framework). Furthermore,
the succession of assignments within the public "reply" method serve to clarify
the process to the (human) reader and also enforce proper temporal succession
by requiring the next method in the sequence to take in as argument the return
value of the previous method (Martin, 2009, pp.302-303). This pattern is repeated
elsewhere throughout the project.

~~~~ {.python}
def reply(self, message):
      userid = message.getUserid()
      messagecontent = self._preprocess(message.getContent())
      reply = self._interpreter.reply(userid, messagecontent)
      reply = self._postprocess(reply)

      return reply
~~~~

The bot_builder module is responsible for creating the concrete instances of
message pre and post processors and their dependencies. Creational duties
are slightly "special" in the sense that they require explicit dependencies on
concrete classes and modules that define the constructors for the objects that
will be used throughout the system. The Dependency Inversion principle states that
the direction of dependency in the core application logic goes towards the abstract
and not the concrete. However, if all our dependencies point at the abstract (which
cannot be instantiated) then how are we going to instantiate them? Hence the need
for creational logic that can limit the number of concrete subclasses of the abstract
type the caller needs to know about.

[^interfaces]: While dynamically typed languages (such as Python) do not require inheritance
for polymorphism, having the interface clearly defined help specify the expectation
to other programmers. Therefore, interfaces are specified and inherited from in the code.

The rivescript_loader module is responsible for loading the chatbot brain files
from the host operating system, and produce errors as needed. This makes it
easier to mock the chatbot framework association of the BotRivescript class.

The façade delegates to a message preprocessor to process the input to the
system and a postprocessor to process the system output. Looking at the
corresponding classes, we notice both of these inherit from the MessageProcessor
class, which is another interface. Looking at the the preprocessor we can see
that it is in turn another façade, like the BotRivescript class: it delegates to
stopword removers, tokenizers and stemmers to normalize and simplify the user
input so that it may be easier to match against the patterns specified in the
chatbot framework grammar. This helps with RiveScript, as certain input patterns
that should be matched do not get matched if the user mispells a word, uses a
declination of a verb or adds
stopwords (such as determinants) which had not been anticipated in the grammar.

For example, in RiveScript, the input pattern in the following rule:

~~~
+ I like you
- I like you too
~~~

would not match the input "I am really liking you",
even though the programmer may have intended for general expressions of affection
to trigger the response. Stemming reduces verbs to their root: "liking" or other
declinations are reduced to "lik". Stopwords removal will eliminate words
that may cause matching to fail, even though they do not modify the meaning of
the sentence significantly, for example "I do like you" (auxilliary verbs are
used in English for emphasis).

Modules responsible for the preprocessing are defined within the "preprocess"
subpackage. The creation of stemmers is handled through a factory that is minimal
for the current implementation, but could in the future include logic to select
stemmers on the basis of a simple parameter input, this would help decouple
the caller from the concrete implementation of the stemming interface (as above).
Notice how the dependency to the nltk package is limited to only the concrete
implementation that makes use of it in this class hierarchy. Just like with the
chatbot framework, the intention is to not tightly couple the core of the system
to the external library, but to keep the coupling loose, so that in the future other
tools or "homebrewed" solutions may be used instead.

More interesting is the application of the TEMPLATE METHOD pattern in the
StopwordRemover abstract class (Gamma et al, 1995, pp.325-330). We can see here how
the stopword remover implemented in the nltk was too strict and would remove
semantically meaningful tokens such as "no", "not" and declinations of "to be"
(presumably on account of the fact that they are also used as auxilliary verbs
  and would create noise in information retrieval tasks).

Therefore, using this stemmer, the input "I do not like you" would have
matched the rule:

~~~
+ I like you
- I like you too
~~~

which may not have been the programmer's intent. Therefore, the stopword removal
algorithm expressed in the StopwordRemover abstract class relies on subclasses
exposing an iterable describing the stopwords to remove from the tokens.

The postprocessor, inheriting from the same MessageProcessor interface as the
preprocessor, is also a façade. The role of the postprocessor in the current
implementation relates to the integration with the search engine being worked
on by Rim Ahsaini.

The modules it defers to apparently extract a keyword from the system output message,
perform a query with that keyword and then decorate the message with the result
before returning this to the main façade. We can see that the postprocessor subsystem
is also defined within its own subpackage, with simple implementations aiming at
extracting a single keyword from the system output message, then putting a single
search query result back into the message.

This system was created ahead of the completion of the external search system,
and was therefore built based on the author's assumptions about the information
required and returned by this other system. The employment of an ADAPTER pattern
sitting at the boundary between the systems enforces compliance with the core
chatbot system's expectations by adapting the expectations of the system on the
other side appropriately, in line with the "plugin model" architecture
(Gamma et al, ibid, pp.139-150; Grenning, 2009, p.119).

This concludes the explanation of the main message system exchange pipeline. This
model allows us to keep the core framework "at arms length", as opposed to
tightly coupling with it and making it almost impossible to replace it in the future,
and in fact it afforded us an intuitive way to overcome the limitations of the framework and
to integrate with an external system. Finally, it exposes a minimal but complete
API to the caller.

### The Brain Implementation

The brain is implemented as a set of RiveScript files, split by categories with the
exception of the begin.rive, the global.rive, and the python.rive files.

These files play each a special role: begin.rive defines the core grammar substitutions and
basic synonyms as arrays, independent of the topic. The global.rive file defines the
global scope, that the other topics inherit from. Finally, the python.rive
file defines the Python macros that can be called from within RiveScript conditionals
and response templates.

We will look at the positives and negatives of the brain implementation in order.

#### The Good

RiveScript provides very useful tools in the ability to call Python (and other
programming languages) from inside reply templates. This enables the programmer
to draw information from outside the interpreter and also perform operations
on behalf of it. This was instrumental in using the external data model provided
by the ConversationDriver data model to drive the conversation forward. Secondly,
RiveScript can store state that is only accessible from within a specific user
session, and manages user sessions internally by requiring that both the user
input and a userId (created by the caller) be passed when asking the RiveScript
for a reply. This is also very useful to store information about the current topic
in the RiveScript. Finally, the topic inheritance model allows us to define
a global scope, wherein we define the matchers that are used to perform topic-invariant
operations (such as changing the topic or moving on to the next concern).

Overall, RiveScript has been a really useful tool to rapidly set up and start
programming a chatbot brain. Unfortunately, many shortcomings were found with it
as the project evolved.

#### The Bad

The first
thing to take into account is that the current implementation of the chatbot brain
is incomplete and not very articulate. It is incomplete because it is at the moment
impossible to move on from any of the topics other than the phyisical issues topic,
therefore any user who has concerns other than physical will never trigger the macro
that changes the state of the DistressConversationDriver data model for the user
to move on to the next topic. It is not articulate because very little time was
spent working on the brain implementation, given the focus of the present project: the system architecture.

The hand written rules that frameworks like RiveScript (and competitors as described
  in Chapter 2) need take a significant amount of time to write and test, more than common programming code.
Secondly, numerous issues were met during development, including textbook use of RiveScript language syntax
constructs causing uncaught exceptions to be thrown from within the RiveScript
class, and issues with inconsistent internal state within the framework internal uservariables that
took three days (amounting to 10% of project core development time) to debug
and fix, due to poorly documented behaviour when changing and using the same
user variable in the breath of the same pattern matcher (including redirects).

Finally, the Python statements in the RiveScript object macros cannot be
extracted to a separate file. This is because the "rs" reference available
therein (which is the running RiveScript interpreter instance) refuses to be
called outside of a .rive file, and will fail with an exception if an attempt is
made to do so. To call this object's properties is necessary to gather information
such as the current user session, and the current state of the user variables.
This makes the statements in the macros untestable.

Due to these shortcomings, the recommendation going forward is to consider using
a different chatbot framework. As mentioned, only a minimal brain implementation
has been provided with the architecture and it would not be an irreparable loss
to start with this aspect of the system from scratch. The reader is redirected
to the discussion of alternatives in Chapter 2.

### Data Models for Concerns and Messages

The concerns and messagelog packages define data models to respectively represent
the user concerns so that these can be accessed from the chatbot brain and used
to decide how to move the conversation forward, and provide a log of conversations
between user and system so that these can be retrieved and stored at a later time.

Additionally, the current implementation of the chatbot requires the argument
to the reply method of the BotRivescript class to be a Message instance. This is
so that the original single argument interface is preserved while both userId
and message content (as required by the underlying RiveScript object) can be
forwarded. This is also so to leave it the caller responsibility to decide
userIds from outside the system, to be able to keep track of the data models.
The user ids, in fact, are used to retrieve the relevant data models (conversation logs,
ConversationDrivers) for all users.

### Categorization

The categorization module does not, for the most part, define classes. It
exposes essentially procedural code, and leverages the facilities of an NLP library
that is built on top of the NLTK: TextBlob (Loria et al, 2016). The reason why
this other package was used for the categorization component is primarily
its capacity to easily split and categorize several sentences, something which
could be achieved with the NLTK, but not with as much abstraction as provided
by TextBlob (https://textblob.readthedocs.io/en/dev/classifiers.html#classifying-textblobs).

The package provides a convenient way to train, serialize, deserialize and
evaluate a single label classifier, but no easy way to swap the concrete
classifier instance and no high level abstraction in the form of an interface or
a façade. This was the product of the author learning Python, becoming more comfortable with
the applicaiton of software design principles and working on the package at the same time.

### Synonym Generation and Automated RiveScript Generation

The final module examined is the synonym generator. The subpackage (as others
  here described) exposes an interface describing the high level abstraction of
the subsystem.

As mentioned in Chapter 2, the implementation here for synonym extraction
was not used in the system delivered by the team because of poor performance.
The model used in the implementation has been trained over a very large data
set of Google News data.  

As it is possible to see from the integration tests (tests/tests_integration/test_synonym/)
the cosin distance does not always appear to capture semantic synonymity (Mikolov, 2015; McCormick, 2016b;
  ). It is
possible to see, for example, that while perfomance may be judged adequate with
respect to words like "dad" or "friend", there are cases when it can only associate
the word with either misspellings or completely irrelevant terms (such as names
  of reporters). This is maybe interesting linguistically, but does not help
with generating synonyms.

As mentioned in the discussion, the implementation of this concrete class was
done with the help of the Gensim library (Rehurek, 2014; Rehurek and Sojka, 2010;
  McCormick, 2016a). The reader is redirected to the discussion of alternatives
in Chapter 2.

As far as generating RiveScript, the syntax of the language is really simple, and
it is not difficult to generate syntactical arrays. The rivescript_writer module
already implements this behaviour. In general for the project going forward,
as long as RiveScript or similar rule-driven chatbot frameworks are used,
it may well be possible to similarly generate data for use in similar ways,
prior exploration of other alternatives to the particular model used in this implementation.

## Conclusion

This concludes the overview of the system design and implementation. We have
looked at the general design principles adopted for the project, the high level
abstractions provided, then the details of the particular implementations provided.
