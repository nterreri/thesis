# System Design and Implementation {#Chapter4}

>> *"Out of all the documentation that software projects normally generate, was
>> there anything that could truly be considered an engineering document?" ...
>> "Yes, there was such a document, and only one--the source code."*[^reeves]

[^reeves]: Reeves, 2001, p.517.

This chapter describes the most interesting aspects of the delivered system
architecture and implementation. First, the design principles followed during
development are described, then a high level overview of the project structure
is provided before moving on to more detailed descriptions of individual
sub-packages.
The overall architecture of the system produced in the team effort
is illustrated below (Figure 4.1).

![System components diagram](componentChatbot.pdf)

## Software Architecture

### Principles of Software Design

SOLID principles of software design were followed in the design of the system;
these are dependency management principles and design principles (Martin, 2003, Section 2).
Additionally, clean code design guidelines (Martin, 2009; McConnell, 2004, chapter 5)
 were also followed.
The chief purpose of these efforts is to design the system for change, and
make the code readable to *humans* (Martin, ibid, pp.13-14).
This approach leads to extremely specialized and short class bodies and functions, and
the separation of different levels of abstraction.
Furthermore, the project was structured with the "plugin model" architecture:
ensuring that the dependencies flow in the direction of the core of the
system (Figure 4.1; Martin, 2012; Martin, 2014).


### High Level Structure

![Internal package structure diagram](architectureChatbot.pdf)

The programmer accessing the package structure
should be able to understand what the purpose of the system and of the
modules within is without further assistance (Martin, 2011).
The internal architecture diagram details the high level blocks of abstraction in the
system (Figure 4.2; Martin, 2003).
We will look at the core message processing logic of the system first, then
discuss the RiveScript brain implementation together with the data
models and conversation management, and finally move to the machine learning components of the system.

## Core Processing

User messages are piped through a preprocessing layer that filters them to simpler
forms in order to simplify the pattern matching in the chatbot brain. The reply
from the brain is also postprocessed in order to decorate reply templates with
results from the search engine when needed. We will look at each layer in detail.

### The BotInterface

![Core processing class diagram](classBotInterfaceChatbot.pdf)

Within the *botinterface* sub-package we can find the core high level
abstractions of the system: the *bot_abstract* module defines the interface
to the bot, with a very restrictive set of methods that
essentially make the abstraction a simple "response machine". Given a message,
an appropriate natural language reply is expected[^interfaces].

The concrete subclass of this interface (*BotRivescript*) is a
FAÇADE: it essentially delegates the processing of the natural language message
to other components of the system (code listing E.1 in Appendix E).
Specifically, it delegates to a message preprocessor to process the input to the
system and a postprocessor to process the system output through a *MessageProcessor*
interface both of these conform to.
The modularity
of the design makes it easy to change implementation of these delegates, and provides
a clear separation of concerns with the message coming into the system
being preprocessed prior to being forwarded to the message interpreter through a
PROXY, and then postprocessed as needed (Gamma et al, 1995, pp.185-193, 207-217; Martin, 2003, pp.327--).

This provides a degree of decoupling from the chatbot
package, instead of making it a central component of the system, allowing it to be changed for
another one of the options surveyed in Chapter 2 with relative ease (see Appendix A.2.1
for more details). Furthermore,
the succession of assignments within the public *reply()* method serve to clarify
the process to the (human) reader and enforce proper temporal succession
by requiring the next method in the sequence to take in as argument the return
value of the previous method (instead of having side-effects
Martin, 2009, pp.302-303). This pattern is repeated
elsewhere throughout the project (see *reply(message)* method in code listing E.1 in Appendix E).

#### BotBuilder

The *bot_builder* module is responsible for creating the concrete instances of
message pre and post processors and their dependencies. Creational duties
are "special" in the sense that they require explicit dependencies on
concrete classes and modules that define the constructors for the objects that
will be used throughout the system (DPI: see Martin, 2003).
To clarify the meaning of "delegates" in Figure 4.3 and other figures in this chapter:
the *BotBuilder* will wire up all of the relevant dependencies, so that the façades
do not have to instantiate the concrete subtypes of the interfaces themselves (Figure 4.2;
see also Appendix A, Section A.2.1).

[^interfaces]: While dynamically typed languages (such as Python) do not require inheritance
for polymorphism, having the interface clearly defined helps specifying the expectation
to other programmers. Therefore, interfaces are specified and inherited from in the code.

### The Preprocessing Layer

![Preprocessor class diagram](classMessagePreprocessorChatbot.pdf)

The preprocessor is in turn another façade,
 like the *BotRivescript* class: it delegates to
stopword removers, tokenizers and stemmers to normalize and simplify the user
input so that it may be easier to match against the patterns specified in the
chatbot framework grammar (Figure 4.4; see code listing E.2 in Appendix E).
This helps with RiveScript, as certain input patterns
that should be matched do not match if the user misspells a word, uses a
declination of a verb or adds
stopwords (such as determiners) which had not been anticipated in the grammar.

Modules responsible for the preprocessing are defined within the "preprocess"
sub-package.
Notice how the dependency to the external NLTK package is limited to only the concrete
implementations that makes use of it in this class hierarchy. Just like with proxying
the rivescript package, the intention is to not tightly couple the core of the system
to the external library, but to keep the coupling loose, so that in the future other
tools or solutions may be used instead.

More interesting is the application of the TEMPLATE METHOD pattern in the
*StopwordRemover* abstract class (Gamma et al, 1995, pp.325-330).
The stopword removal
algorithm expressed in the *StopwordRemover* abstract class relies on subclasses
exposing an iterable defining the stopwords to remove from the tokens.
The set of stopwords defined in the NLTK is too strict and would remove
semantically meaningful tokens such as "no" and declinations of "to be"
(presumably on account of the fact that these would create noise in some
information retrieval tasks). Therefore, a more lenient implementation was
provided and used in the system.

### The Postprocessing Layer

![Postprocessor class diagram](classMessagePostprocessorChatbot.pdf)

The postprocessor, inheriting from the same *MessageProcessor* interface as the
preprocessor, is also a façade. The role of the postprocessor in the current
implementation relates to the integration with the external search engine component
(Figure 4.5; code listing E.3 Appendix E).

The modules it defers to extract keywords from the system output message,
perform a query with the keywords and then decorate the message with the results
before returning this to the main façade. We can see that the postprocessor subsystem
is also defined within its own sub-package, currently using simple implementations aiming at
extracting a single keyword from the system output message, then putting a single
search query result back into the message[^notrivescript].

[^notrivescript]: Note that RiveScript can achieve the same behaviour via macros.
However, this approach decouples the integration with the search engine from
the chatbot package of choice (it does not matter if it is replaced in the future).
Furthermore, it affords more control: the decorated system reply may be a
complex object made of a message content and list of search results for the
caller to handle as they prefer.

This system was created ahead of the completion of the external search system,
and was therefore built based on the author's assumptions about the information
required and returned by this other system. The employment of an ADAPTER pattern
sitting at the boundary between the systems enforces compliance with the core
chatbot system's expectations by adapting the expectations of the system on the
other side appropriately, in line with the "plugin model" architecture
(Gamma et al, ibid, pp.139-150; Grenning, 2009, p.119; Figure 4.1).

This concludes the explanation of the main message system exchange pipeline. This
model allows us to keep the external tools "at arm's length".
The design also afforded us an intuitive way to overcome the limitations of RiveScript and
to integrate with an external system. Finally, it exposes a simple
API to the caller.

## The Brain and Data Models

We will now discuss how the brain interfaces with the conversation management logic,
and then briefly discuss how user concerns are modelled before moving on to
discussing how messages and conversations are modelled (Figure 4.2).

### The Brain

![Conversation management diagram](macrosChatbot.pdf)

The *brain/* folder contains the RiveScript language files.
These define the matchable
patterns, reply templates
and topics structure as seen from the brain. Topics limit the scope of pattern
matchers depending on the stage of the conversation between user and system,
allowing only responses pertinent to the topic to be returned to the user.

The *begin.rive*, the *global.rive*, and the *python.rive*
files play each a special role: *begin.rive* defines basic grammar substitutions and
basic synonyms as arrays. The *global.rive* file defines the
global scope that the other topics inherit from. Finally, the *python.rive*
file defines the Python macros that can be called from within RiveScript conditionals
and response templates (Figure 4.6, code listings E.9, E.10 Appendix E).

### Conversation Management and Concern Models

![Concerns class diagram](classConcerns.pdf)

The *concerns* package define data models to represent
the user concerns so that these can be queried by the brain to decide how to
move the conversation forward. The conversation concern logic
is accessed through Python object macros written into the *python.rive* brain file (see above).

The Python statements in these macros import the *rivescriptmacros* module (Figure 4.6). This module
communicates with the *ConversationDriver* interface, which decides
what topic to talk about next, and whether a query via the search engine
should be made (Figure 4.7). This is accessed through a static *UserConcernsFactory* module
that lazily instantiates a *ConversationDriver* for each different user ID. Each
*ConversationDriver* instance possesses
a record of concerns for the user that it uses to make decisions[^distress].

[^distress]: Concretely,
these are *DistressConversationDriver* instances that
keep track of what the user concerns are and their priority
order based on distress scores.

Finally, the topics package contains hard-coded information about the relation
of microtopics to macrotopics, inferred from the CC (see Chapter 2). This is
used to control the internal state of the brain with respect to conversations
with users, changing the available set of pattern matchers based on the macrotopic
being discussed and on the user ID received with the message.

### Conversation and Message Models

![Conversation class diagram](classMessagelog.pdf)

The *messagelog* package defines very simple message logging facilities, primarily
for use in the future to be stored durably (Figure 4.1; Figure 4.8).

Additionally, the current implementation of the chatbot requires the argument
to the reply method of the *BotRivescript* class to be a *Message* instance. This is
so that the original single argument interface is preserved while both user ID
and message content (as required by the underlying *RiveScript* object; see code
listing E.4 Appendix E) can be forwarded. This is also so to leave it the caller
responsibility to decide user IDs from outside the chatbot system, to be able to keep
track of the data models. The user ids, in fact, are used to retrieve the
relevant data models (conversation logs, concerns) for all users
(see listing E.8 Appendix E for example of use).

## Machine Learning

Finally, we look at the parts of the system that leverage machine learning (Figure 4.2). These
are the categorizer and the synonym generator discussed in Chapter 2.

### Categorization

The categorization module does not, for the most part, define classes. It
exposes essentially procedural code, and leverages the facilities of an NLP library
that is built on top of the NLTK: TextBlob (Loria et al, 2016). The reason why
this other package was used for the categorization component instead of just the NLTK is primarily
its capacity to easily deserialize a classifier and the facilities it exposes to
classify several distinct sentences within a single *String*, which could
prove useful when dealing with a single user input that consists of a sequence of
distinct sentences[^sentences].

[^sentences]: <https://textblob.readthedocs.io/en/dev/classifiers.html#classifying-textblobs>.

The categorization package implemented provides a way to train, serialize, deserialize and
evaluate a single label classifier, but no easy way to swap the concrete
classifier instance and no high level abstraction in the form of an interface or
a façade. This package is
the least well designed element of the system because it was the first to be
created. This was the product of the author
learning Python, learning about NLP and becoming more comfortable with
the application of software design principles while working on the system at the
same time.

The reason this sub-package was not used in the final pipeline
implementation is that it took too long for the survey
that was created to collect relevant data to produce results.
But it is possible
in principle to plug a trained categorizer at the preprocessor level of the
message pipeline in order to add a tag to the user message, in order for this
to be used by the chatbot framework to provide better replies to the user,
as originally intended (see Chapter 2, Chapter 5; and Figure 4.4).

### Synonym Generation

![Synonym generator class diagram](classSynonym.pdf)

The final package examined is the synonym generator (Figure 4.9).
The purpose for this package is to aid in the implementation of
a RiveScript brain by avoiding having to manually define common synonyms to simplify
the pattern matchers in the brain[^brainexample].

[^brainexample]: For example, in the following pattern we refer to arrays in order
to capture semantically equivalent ways to express the desire to change topic
(see also listing E.10 Appendix E):
\+ [*] (\@desire) (\@discuss) (\@other) [\@topic]

The *SynonymExtractorFactory* lazily instantiates a single pre-trained word2vec
language model through the use of the Gensim library (McCormick, 2016a; Rehurek and
  Sojka, 2010).
This can then be asked to produce synonyms for a given word.
The synonyms produced are then used to produce RiveScript arrays.

These are represented as Python objects by the *RivescriptArray* class,
the \__str__() method (equivalent to Java's toString())
of which is used by the *RivescriptArrayWriter* to write syntactically correct
RiveScript arrays to file[^streaming].

[^streaming]: There are also ways to stream RiveScript code into a running
interpreter, see:
     <http://rivescript.readthedocs.io/en/latest/rivescript.html#rivescript.rivescript.RiveScript.stream>.

## Conclusion

This concludes the overview of the system design and implementation. We have
looked at the general design principles adopted for the project, the high level
abstractions provided, and the details of the particular subsystems in the
 implementation.
