# System Design and Implementation {#Chapter4}

> *"Out of all the documentation that software projects normally generate, was
> there anything that could truly be considered an eningeering document?"* The
> answer that came to me was, *"Yes, there was such a document, and only one--the source code."*

**J. Reeves, 2001**

This chapter describes the most interesting aspects of the delivered system
architecture and implementation. First, the design principles followed during
development are described, then a high level overview of the project structure
is provided before moving on to more detailed descriptions of individual
subpackages.

As mentioned before, the complete application is the product of a team effort,
a web server and specialized search engine being separately developed, while the author focuses
on the core chatbot brain. The overall architecture of the components is illustrated
below (Figure 4.1).

![System component diagram](componentChatbot.pdf)

## Software Architecture

### Principles of Software Design

SOLID principles of software design were followed in the design of the system;
these are dependency management principles (policing the "import" statements
throughout the project) and clean design principles, chief among them the Separation
of Concerns principle (Martin, 2003, Section 2).
Additionally, the clean code design guidelines (Martin, 2009) were also followed.
The chief purpose of these efforts is to design the system for change, and
make the code readable to *humans* (Martin, ibid, pp.13-14).
This approach leads to extremely specialized and short class bodies and functions, and
the separation of different levels of abstraction.
Additionally, the project was structured with the "plugin model" architecture:
ensuring that the direction of dependency flow in the direction of the core of the
system (Figure 4.1).

The core of a chatbot system are the high level abstractions precisely defined in
the interfaces and the abstract classes (Dependency Inversion Principle: Martin, 2003)

### High Level Structure

![Architecture diagram](architectureChatbot.pdf)

The programmer accessing the package structure
should be able to understand what the purpose of the system and of the
modules within is without further assistance (see Martin, 2011).

The architecture diagram details the high level blocks of abstraction in the
system (Figure 4.2).
We will look at the core message processing logic of the system first, then
discuss the RiveScript brain implementation,
then move on to the data
models and conversation management, and finally the machine learning components of the system.

## Core Processing

User messages are piped through a preprocessing layer that filters them to simpler
forms in order to simplify the pattern matching in the chatbot brain. The reply
from the brain is also postprocessed in order to decorate reply templates with
results from the search engine when needed. We will look at each layer in detail.

### The BotInterface

![Core processing class diagram](classBotInterfaceChatbot.pdf)

Within the botinterface subpackage we can find the core high level
abstractions of the system: the bot_abstract module defines the interface
to the bot, with a very restrictive set of methods that
essentially make the abstraction a simple "response machine". Given a message,
an appropriate natural language reply is expected[^interfaces].

Looking at the the concrete subclass of this interface, BotRivescript, we find it is in fact a
FAÇADE: it essentially delegates the processing of the natural language message
to other components of the system.
Specifically, it delegates to a message preprocessor to process the input to the
system and a postprocessor to process the system output through a MessageProcessor
interface both of these conform to.
The modularity
of the design makes it easy to change implementation of these delegates, and provides
a clear separation of concerns with the message coming into the system
being preprocessed prior to being forwarded to the message interpreter through a
PROXY, and then postprocessed as needed (Gamma et al, 1995, pp.185-193, 207-217; Martin, 2003, pp.327--).

This provides a degree of decoupling from the chatbot
package, instead of making it a central component of the system, allowing it to be changed for
another one of the options surveyed in Chapter 2 with relative ease (see Appendix A
for more details). Furthermore,
the succession of assignments within the public "reply" method serve to clarify
the process to the (human) reader and also enforce proper temporal succession
by requiring the next method in the sequence to take in as argument the return
value of the previous method (Martin, 2009, pp.302-303). This pattern is repeated
elsewhere throughout the project:

~~~~ {.python}
#from botinterface/bot_rivescript.py
def reply(self, message):
      userid = message.getUserid()
      messagecontent = self._preprocess(message.getContent())
      reply = self._interpreter.reply(userid, messagecontent)
      reply = self._postprocess(reply)

      return reply
~~~~

#### BotBuilder

The bot_builder module is responsible for creating the concrete instances of
message pre and post processors and their dependencies. Creational duties
are slightly "special" in the sense that they require explicit dependencies on
concrete classes and modules that define the constructors for the objects that
will be used throughout the system (Dependency Inversion Principle, see Martin, 2003).
To clarify the meaning of "delegates" in Figure 4.2 and others in this chapter:
the BotBuilder will wire up all of the relevant dependencies, so that the façades
do not have to instantiate the concrete subtypes of the interfaces themselves.

[^interfaces]: While dynamically typed languages (such as Python) do not require inheritance
for polymorphism, having the interface clearly defined help specify the expectation
to other programmers. Therefore, interfaces are specified and inherited from in the code.

### The Preprocessing Layer

![Preprocessor class diagram](classMessagePreprocessorChatbot.pdf)

The preprocessor is in turn another façade,
 like the BotRivescript class: it delegates to
stopword removers, tokenizers and stemmers to normalize and simplify the user
input so that it may be easier to match against the patterns specified in the
chatbot framework grammar (Figure 4.4). This helps with RiveScript, as certain input patterns
that should be matched do not match if the user mispells a word, uses a
declination of a verb or adds
stopwords (such as determiners) which had not been anticipated in the grammar.

Modules responsible for the preprocessing are defined within the "preprocess"
subpackage.
Notice how the dependency to the external nltk package is limited to only the concrete
implementations that makes use of it in this class hierarchy. Just like with proxying
the chatbot package, the intention is to not tightly couple the core of the system
to the external library, but to keep the coupling loose, so that in the future other
tools or solutions may be used instead.

More interesting is the application of the TEMPLATE METHOD pattern in the
StopwordRemover abstract class (Gamma et al, 1995, pp.325-330).
The stopword removal
algorithm expressed in the StopwordRemover abstract class relies on subclasses
exposing an iterable defining the stopwords to remove from the tokens.
The set of stopwords defined in the nltk is too strict and would remove
semantically meaningful tokens such as "no", "not" and declinations of "to be"
(presumably on account of the fact that this is also used as an auxilliary verb
  and would create noise in information retrieval tasks).
Therefore, a more lenient implementation was provided and used in the system.

### The Postprocessing Layer

![Postprocessor class diagram](classMessagePostprocessorChatbot.pdf)

The postprocessor, inheriting from the same MessageProcessor interface as the
preprocessor, is also a façade. The role of the postprocessor in the current
implementation relates to the integration with the external search engine component
(Figure 4.5).

The modules it defers to extract a keyword from the system output message,
perform a query with that keyword and then decorate the message with the result
before returning this to the main façade. We can see that the postprocessor subsystem
is also defined within its own subpackage, currently using simple implementations aiming at
extracting a single keyword from the system output message, then putting a single
search query result back into the message.

This system was created ahead of the completion of the external search system,
and was therefore built based on the author's assumptions about the information
required and returned by this other system. The employment of an ADAPTER pattern
sitting at the boundary between the systems enforces compliance with the core
chatbot system's expectations by adapting the expectations of the system on the
other side appropriately, in line with the "plugin model" architecture
(Gamma et al, ibid, pp.139-150; Grenning, 2009, p.119; Figure 4.1).

This concludes the explanation of the main message system exchange pipeline. This
model allows us to keep the external tools "at arms length", as opposed to
tightly coupling with it and making it almost impossible to replace it in the future.
The design also afforded us an intuitive way to overcome the limitations of RiveScript and
to integrate with an external system. Finally, it exposes a simple
API to the caller.

## The Brain and Data Models

We will now discuss how the brain interfaces with the conversation management logic,
and then briefly discuss how user concerns are modelled before moving on to
discussing how messages and conversations are modelled (Figure 4.2).

### The Brain

![Conversation management diagram](macrosChatbot.pdf)

The brain folder contains the RiveScript language files.
These define the matchable
patterns and reply templates
and the topics structure as seen from the brain. Topics limit the scope of pattern
matchers depending on the the stage of the conversation between user and system,
allowing only responses pertinent to the topic to be returned to the user.

The begin.rive, the global.rive, and the python.rive
files play each a special role: begin.rive defines basic grammar substitutions and
basic synonyms as arrays. The global.rive file defines the
global scope, that the other topics inherit from. Finally, the python.rive
file defines the Python macros that can be called from within RiveScript conditionals
and response templates.

### Conversation Management and Concern Models

![Concerns class diagram](classConcerns.pdf)

The concerns package define data models to represent
the user concerns so that these can be be queried by the brain to decide how to
move the conversation forward. The conversation concern logic
is accessed through Python object macros written into the python.rive brain file (see above).

The Python statements in these macros import the rivescriptmacros module (Figure 4.6). This module
communicates with the ConversationDriver interface, which decides
what topic to talk about next, and whether a query via the search engine
should be made (Figure 4.7). This is accessed through a static UserConcernsFactory module
that lazily instantiates a ConversationDriver for each different userid. Each
ConversationDriver instance possesses
a record of concerns for the user that it uses to make decisions[^distress].

[^distress]: Concretely, in the implementation provided,
these are DistressConversationDriver instances that
keep track of what the user concerns are and their priority
order based on distress scores.

Finally, the topics package contains hard-coded information about the relation
of microtopics to macrotopics, inferred from the CC (see Chapter 2). This is
used to control the internal state of the brain with respect to conversations
with users, changing the available set of pattern matchers based on the macrotopic
being discussed.

### Conversation and Message Models

![Conversation class diagram](classMessagelog.pdf)

The messagelog package defines very simple message logging facilities, primarily
for use in the future to be stored durably (Figure 4.1; Figure 4.8).

Additionally, the current implementation of the chatbot requires the argument
to the reply method of the BotRivescript class to be a Message instance. This is
so that the original single argument interface is preserved while both userId
and message content (as required by the underlying RiveScript object) can be
forwarded. This is also so to leave it the caller responsibility to decide
userIds from outside the system, to be able to keep track of the data models.
The user ids, in fact, are used to retrieve the relevant data models (conversation logs,
concerns) for all users.

## Machine Learning

Finally, we look at the parts of the system that leverage machine learning (Figure 4.2). These
are the categorizer and the synonym generator discussed in Chapter 2.

### Categorization

The categorization module does not, for the most part, define classes. It
exposes essentially procedural code, and leverages the facilities of an NLP library
that is built on top of the NLTK: TextBlob (Loria et al, 2016). The reason why
this other package was used for the categorization component instead of just the NLTK is primarily
its capacity to easily deserialize a classifier and the facilities it exposes to
classify several distinct sentences within a single String, which could
prove useful when dealing with a single user input that consists of a sequence of
distinct sentences[^sentences].

[^sentences]: <https://textblob.readthedocs.io/en/dev/classifiers.html#classifying-textblobs>

The package provides a way to train, serialize, deserialize and
evaluate a single label classifier, but no easy way to swap the concrete
classifier instance and no high level abstraction in the form of an interface or
a façade. This package is
the least well designed element of the system: this was the product of the author
learning Python, becoming more comfortable with
the applicaiton of software design principles while working on the system at the
same time.

The reason this subpackage was not used in the final pipeline
implementation is that it took too long for the survey
that was created to collect relevant data to produce results (#REFERENCERELEVANTPART). But it is possible
in principle to plug a trained categorizer at the preprocessor level of the
message pipeline in order to add a tag to the user message, in order for this
to be used by the chatbot framework to provide better replies to the user,
as originally intented (see Chapter 2; and Figure 4.4).

### Synonym Generation

![Synonym generator class diagram](classSynonym.pdf)

The final package examined is the synonym generator (Figure 4.9).
The use case for this package is to aid in the construction of
a RiveScript brain by avoiding having to manually define common synonyms to simplify
the pattern matchers in the brain[^brainexample].

[^brainexample]: For example, in the following pattern we refer to arrays in order
to capture semantically equivalent ways to express the desire to change topic:
\+ [*] (\@desire) (\@discuss) (\@other) [\@topic]

The SynonymExtractorFactory lazily instantiates a single pre-trained word2vec
language model through the use of the gensim library.
This can then be asked to produce synonyms for a given word.
The synonyms produced are then used to produce RiveScript arrays.

These are represented in Python by the RivescriptArray class, the *\__str__()* method
of which is used by the RivescriptArrayWriter to write syntactically correct
RiveScript arrays to file.

## Conclusion

This concludes the overview of the system design and implementation. We have
looked at the general design principles adopted for the project, the high level
abstractions provided, then the details of the particular subsystems in the
 implementation provided.
