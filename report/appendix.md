\appendix

# System Manual

As stated before, the scope of this report is limited to the core chatbot system
which interfaces with a webserver for delivery to
a user in the larger team effor. For this reason, no user manual is provided.

## Source Code

The full project source code
is available at: <https://github.com/nterreri/peach-bot>.

It can also be found under the PEACH chatbot team project repository
under the '/FlaskWebProject/chatbot/' directory therein, together with the code
produced by the rest of the PEACH chatbot team
 (<https://github.com/andreallorerung/peach-chatbot-alpha/tree/master/FlaskWebProject/chatbot>).

In order to obtain the code it is possible to download the repository as a zip
file from the relevant github.com webpage, or use the git software to clone into the
repository (*$ git clone https://github.com/andreallorerung/peach-chatbot-alpha*). The
categorizer package is provided as a git submodule to the chatbot package.
Cloning or downloading the repository will not automatically download the submodule
content. To download all submodules when cloning, use
*$ git clone --recursive https://github.com/andreallorerung/peach-chatbot-alpha*.
It is not possible to do the same if downloading the repository as a zip file,
which means it is necessary to navigate manually to the categorizer repository
(<https://github.com/nterreri/peach-categorize>) before downloading it[^nonrecursive].

The project is meant to be compiled through a Python 2.7 interpreter, no support
is provided for Python 3.x. This is primarily because of the preference of memebers
of PEACH. The project has been only compiled and tested using the default C
Python compiler which should come by default with any official Python distribution.

[^nonrecursive]: When wanting to download the submodule after having cloned into the repository
non-recursively, use *$ git submodule update --init --recursive* to download
the submodule.

### Dependencies

The project uses the following Python packages:

~~~
gensim==0.13.1
nltk==3.2.1
pytest==2.9.2
pytest-cov==2.3.1
rivescript==1.14.1
textblob==0.11.1
~~~

These in turn depend on should be automatically installed when
installing them via the *pip* utility[^pip]. The most straightforward way
to install the dependencies is to use
*pip install -r FlaskWebProject/chatbot/requirements.txt*, but the dependencies
may also be installed piecemeal.

[^pip]: <https://en.wikipedia.org/wiki/Pip_(package_manager)> Python Software Foundation, 2016

## Package Structure

The chatbot Python package contains several subpackages (directories) diving up the software
in large abstractions:

~~~
chatbot/
|-- botinterface/
|-- brain/
|-- categorizer/
|-- concerns/
|-- messagelog/
|-- postprocess/
|-- preprocess/
|-- synonym/
`-- tests/
~~~

### The Chatbot

The *botinterface/* package exposes the interface class for the core chatbot
component in the *botinterface/bot_interface.py*, and provides an implementation
in *botinterface/bot_rivescript.py*. In order to depend only on the API, it is
possible to ask the *bot_builder.py* module (located in the project top level directory)
to construct a concrete subtype of the interface.

This last module exposes a *build()* method that will return a default instance
of the bot, which will assume the RiveScript brain files are located within
a *brain/* directory immediately accessible. The same
package defines a *BotBuilder* class that the caller can use to customize the
chatbot instance being returned. For example, the following code listing will
initialize a chatbot with a brain located at the specified filepath ("path/to/brain"):

~~~ python
botBuilder = bot_builder.BotBuilder()
botBuilder.addBrain("path/to/brain/")
chatbot = botBuilder.build()
~~~

The RiveScript-based chatbot instance returned expects messages to be forwarded
to it in the form of a *messagelog.message.Message* instance. The Message class
simply requires that both a userid and message content be provided. This is
to enforce conformity with the interface defined in the *botinterface/bot_abstract.py*
module: the *reply(message)* method should take a single argument while the caller
retains control of the creation and management of userids independently of
the core chatbot component (Figure 4.1).

As mentioned before, the conversation logging facilities provided by the
*messagelog* package are not automatically used by the chatbot instance, instead
something like code listing (#SOMENUMBER) is necessary.

#### Why RiveScript should be removed

Overall, RiveScript has been a really useful tool to rapidly set up and start
programming a chatbot brain. Unfortunately, many shortcomings were found with it
as the project evolved.

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

#### Replacing RiveScript

In order to replace the RiveScript interpreter, it is necessary to create another
implementation of the Interpreter class, adapting the implementation
provided in RivescriptProxy.
Depending on the framework used, it may be necessary
to create a proxy for a unit of software implemented in other languages than
Python. This is made easier by the various Python compilers that allow other
languages to be used together with Python[^hitchhike].

Effectively, the BotRivescript class only expects its interpreter to expose a
"reply" method, but other aspects of the implementation may change as well such as
how to start the conversation with the user, all changes necessary should however
be confined to the new proxy (see also Figure 4.3).

[^hitchhike]: See Reitz and Schlusser, 2016
here: <http://docs.python-guide.org/en/latest/starting/which-python/>

Requirements for the pre- and postprocessing of messages are also likely to change,
therefore similar considerations apply to their packages.
Finally, it would be necessary to alter the *bot_builder* module to include the
new or modified implementation, as
this would need to know and name (import) the new implementations.

See Chapter 2 for a discussion of alternatives to RiveScript.

### Categorizer

The categorizer can be used through the *demo.py* module. Effectively, this module
should in the future become a more complete utility callable from the command
line, that would train or load a serialized classifier before running it against
a development or test set. As it stands, running the module will always result in
the classifier being trained, run against the dev set and finally tested automatically.

### Synonym Generation

The synonym generation facilities are meant to be accessed through the
*synonym/synonym_extractor_factory.py* module. This module lazily instantiates
an instance of the *Word2VecSynonymExtractor* class (defined in *synonym/synonym_word2vecextractor.py*).

Instantiating this object requires will take up around 6-8 GBs of main memory,
due to the size of the pretrained data model it uses. After it has been loaded,
it is possible to obtain synonyms from it given a word. Doing this may cause
a lot of pages to be swapped in and out of virtual memory
as the model retrieves similar words based on cosin distances. Sample
usage of this module (adapted from */tests/tests_integration/test_synonym/test_synonymmodelfactory.py*):

~~~ python
synonymsFor = dict()
for word in WORDS_TO_GET_SYNONYMS_FOR:
      synonymsExtracted = extractor.extractSynonyms(word)

      synonymsFor[word] = synonymsExtracted
~~~

This will return a list of words which the model believes are synonyms of the
argument to the *extractSynonyms()* method. As described in Appendix C, this
particular implementation was underperforming, but provides
the blueprint for future better performing implementations (see Chapter 2 for
a discussion of alternatives).

## Full Concerns Checlist

As mentioned in Chapter 2, the full concerns checklist is presented here for
reference, these are encoded in the *concerns/topics.py* module:

~~~
Physical concerns 1         Physical concerns 2
- Breathing difficulties    - Sore or dry mouth
- Passing urine             - Nausea or vomiting
- Constipation              - Sleep problems/nightmares
- Diarrhoea                 - Tired/exhausted or fatigued
- Eating or appetite        - Swollen tummy or limb
- Indigestion               - High temperature or fever

Physical concerns 3         Physical concerns 4
- Getting around (walking)  - Changes in weight
- Tingling in hands/feet    - Memory or concentration
- Pain                      - Taste/sight/hearing
- Hot flushes/sweating      - Speech problems
- Dry, itchy or sore skin   - My appearance
- Wound care after surgery  - Sexuality

Practical concerns 1        Practical concerns 2
- Caring responsibilities   - Housework or shopping
- Work and education        - Washing and dressing
- Money or housing          - Preparing meals/drinks
- Insurance and travel
- Transport or parking
- Contact/communication
    with NHS staff

Family concerns             Emotional concerns 1
- Partners                  - Difficulty making plans
- Children                  - Loss of interest/activities
- Other relatives/friends   - Unable to express feelings
                            - Anger or frustration
                            - Guilt
                            - Hopelessness

Emotional concerns 2        Spiritual or religious concerns
- Loneliness or isolation   - Loss of faith or other spiritual concerns
- Sadness or depression     - Loss of meaning or purpose of life
- Worry, fear or anxiety    - Not being at peace with or feeling regret about
                                the past
~~~

## Running py.test

The tests included are meant to be run using py.test (Krekel, 2016).
In order to allow the required project packages to be imported, this should be
run from the project top level directory (the one overseeing all the subpackages
and test folder). The directory or test file to run can be specified as an
argument to the Python CLI interpreter:

~~~
python -m py.test tests/
~~~

Running this command will run all the tests in the package, which on a fairly
powerful machine can take up to 5 minutes, but can potentially make a slower
machine hang as for an unreasonable amount of time. This is chiefly due to the
synonym generation tests which require a very large amount of RAM to run efficiently
(see above).

It is good to notice at this point the difference in runtime between the decoupled unit tests
and the integration tests in this case, running all unit tests for the project takes
mere seconds (as it should).

The unit tests, in fact, are designed for quick execution (see Martin, 2009, p.132).
Running these tests should take less than a minute.
Finally, if the pytest-cov package is been installed (Schlaich, 2016) then it
is possible to get reports for code coverage about individual subpackages
by providing the desired subpackage name as an argument in the following manner:

~~~
python -m py.test --cov=botinterface tests/tests_unit/
~~~

This will provide an executable statement test coverage report.
The reader is redirected to Chapter 5 for more
information about the tests and Appendix B for a collected test coverage
report.

# Unit Tests Results

This appendix reports the unit testing code coverage for all modules in the
project. These reports are taken from the pytest CLI because pytest-cov can only
write HTML, XML or annotated source code to file, but not plain text as shown here.
All tests were, of course, made to pass. Test coverage data follows[^legend]:

[^legend]: Legend: "Stmts" stands for total number of executable statements found in
module, "Miss" for statements not executed by any test, "Cover" is the percentage
of statements covered by the tests.

\rule{14.75cm}{0.4pt}
~~~
---------- coverage: platform cygwin, python 2.7.10-final-0 ----------
Name                                    Stmts   Miss  Cover
-----------------------------------------------------------
botinterface/__init__.py                    0      0   100%
botinterface/bot_abstract.py                7      3    57%
botinterface/bot_builder.py                23      5    78%
botinterface/bot_rivescript.py             30      1    97%
botinterface/message_processor.py           5      2    60%
botinterface/postprocessor.py              21      0   100%
botinterface/postprocessor_example.py      20      0   100%
botinterface/preprocessor.py               20      0   100%
botinterface/rivescript_loader.py          13      1    92%
categorize/__init__.py                      0      0   100%
categorize/classifierDeserializer.py        5      0   100%
categorize/classifierSerializer.py          4      0   100%
categorize/dataset_reading.py              12      4    67%
categorize/dataset_splitting.py            10      0   100%
categorize/develop.py                      10     10     0%
categorize/evaluation.py                   46      2    96%
categorize/training.py                      4      0   100%
concerns/__init__.py                        0      0   100%
concerns/concern.py                        10      0   100%
concerns/concern_factory.py                13      0   100%
concerns/drive_conversation.py             54      0   100%
concerns/drive_conversation_abstract.py    11      5    55%
concerns/equivalence.py                    32      1    97%
concerns/rivescriptmacros.py               24      3    88%
concerns/topics.py                         14      0   100%
messagelog/__init__.py                      0      0   100%
messagelog/conversation.py                  6      0   100%
messagelog/conversation_logging.py         21      0   100%
messagelog/message.py                       8      0   100%
postprocess/__init__.py                     0      0   100%
postprocess/keyword_extractor.py            5      2    60%
postprocess/keyword_extractor_single.py    30      0   100%
postprocess/message_decorator.py            5      2    60%
postprocess/message_decorator_single.py    29      0   100%
postprocess/postprocessor_builder.py        5      0   100%
postprocess/search_adapter.py               5      2    60%
preprocess/__init__.py                      0      0   100%
preprocess/preprocessor_builder.py          6      0   100%
preprocess/stemmer_factory.py               3      0   100%
preprocess/stemming.py                      5      2    60%
preprocess/stemming_lancaster.py            7      1    86%
preprocess/stopword_remover_nltk.py         6      0   100%
preprocess/stopwords_remover.py            17      1    94%
preprocess/stopwords_remover_lenient.py     5      0   100%
preprocess/tokenizer.py                     5      2    60%
preprocess/tokenizer_simple.py              7      1    86%
synonym/__init__.py                         0      0   100%
synonym/rivescript_array.py                 9      0   100%
synonym/rivescript_writer.py                7      7     0%
synonym/store_synonyms.py                   4      4     0%
synonym/synonym_extractor.py                5      2    60%
synonym/synonym_extractor_factory.py       13      5    62%
synonym/synonym_word2vecextractor.py       17      0   100%
----------------------------------------------------------
TOTAL                                     527     52    89%
~~~

Note that there are several abstract classes that are also counted in the total
executable statements even though they should not, for example *MessageProcessor*:

~~~ python
#botinterface/message_processor.py
class MessageProcessor(object):
    '''Interface a message content processor is expected to implement'''
    def __init__(self):
        raise NotImplementedError("MessageProcessor is an interface")

    def process(self, sentence):
        '''To preprocess the sentence (content of a message)'''
        raise NotImplementedError("MessageProcessor is an interface")

~~~

The two "raise" statements are counted when they should not be, similarly with
many other classes. This means that the actual test coverage is slightly higher.
There are 27 statements from interfaces[^abstract] that should not
be counted as statements at all, bringing the metrics to 500 total statements
and 25 missed statements bringing coverage to 95%.

[^abstract]: Specifically, the interfaces: *botinterface/bot_abstract.py*,
*botinterface/message_processor.py*, *concerns/drive_conversation_abstract.py*,
*postprocess/keyword_extractor.py*, *postprocess/message_decorator.py*,
*postprocess/search_adapter.py*, *preprocess/stemming.py*, *preprocess/tokenizer.py*,
*synonym/store_synonyms.py*, *synonym/synonym_extractor.py*, finally the constructor
of the abstract class *preprocess/stopwords_remover.py* should also not be counted.

There are also integration tests testing the interaction of two
or more independent units, or other aspects of the system that do not belong to
unit testing, such as using the actual word2vec model instead of a mock. Again,
all such tests were made to pass. As mentioned in Chapter 5, the RiveScript brain
logic is also extensively tested via integration tests.

Finally, the RiveScript Python macro code within *brain/python.rive*
is not considered and not directly covered by any tests and is in fact
impossible to test in isolation. This is because
the RiveScript object instance "rs" accessible within these
macros cannot be called from inside separate and isolated methods (as it was the
author's original intention) and will raise an error if it is passed as a variable
to an externally defined unit of Pyhon code. This aspect of RiveScript is poorly documented, and
another reason to move away from the framework in the future (Petherbridge, 2009,
    <https://www.rivescript.com/wd/RiveScript#OBJECT-MACROS>).

# Machine Learning Evaluation

In this section, detailed evaluation results are provided for the categorization
task, and a summary evaluation is provided for the synonym generation task.

## Categorizer Evaluation

The survey gathered 222 data points in total (see Chapter 2). The same split was used for the
training and testing of both classifier implementations used.
The single-label categorizers used in the project are provided by
the nltk through the TextBlob package. A Naive Bayes classifier and
a Maximum Entropy classifier.

The data splitting logic in the *categorize* package by default simply takes
10% of the labelled data set for training, and another 10% for developement:

~~~ python
#categorize/dataset_splitting.py
def split(labelled_data, testsizepercent = 0.1, devsizepercent = 0.1, \
          shuffle = False):
~~~

The results of running the Naive Bayes classifier against the test set
(around 22 test cases) is disappointing:

\rule{14.75cm}{0.4pt}
~~~
Accuracy: 0.36
Macro averaged recall: 0.29
Macro averaged precision: 0.35
Micro averaged recall: 0.36
Micro averaged precision: 0.36

Recall for 'pract': 0.0         Recall for 'phys': 0.67
Precision for 'pract': 0.0      Precision for 'phys': 0.33
F for 'pract': 0                F for 'phys': 0.44
Recall for 'family': 0.4        Recall for 'emotion': 0.4
Precision for 'family': 1.0     Precision for 'emotion': 0.4
F for 'family': 0.57            F for 'emotion': 0.4
Recall for 'spiritual': 0.0
Precision for 'spiritual': 0.0
F for 'spiritual': 0
~~~
\rule{14.75cm}{0.4pt}

All metrics for the 'pract' and 'spiritual' labels are 0 which means the classifier
systematically gets them wrong over the data set. The micro and macro averages
of precision and recall for all labels score around 33%.

The MaxEntClassifier, instead, scores over the same data split (10% test, 10% dev,
  80% train) when made to iterate over the dev set for 10 iterations:

\rule{14.75cm}{0.4pt}
~~~
Accuracy: 0.55
Macro averaged recall: 0.4467
Macro averaged precision: 0.4911
Micro averaged recall: 0.5454
Micro averaged precision: 0.5454

Recall for 'pract': 0.4         Recall for 'phys': 0.83
Precision for 'pract': 0.4      Precision for 'phys': 0.56
F for 'pract': 0.4              F for 'phys': 0.67
Recall for 'family': 0.4        Recall for 'emotion': 0.6
Precision for 'family': 1.0     Precision for 'emotion': 0.5
F for 'family': 0.57            F for 'emotion': 0.55
Recall for 'spiritual': 0.0
Precision for 'spiritual': None
F for 'spiritual': None
~~~
\rule{14.75cm}{0.4pt}

Like the Naive Bayes classifier, it scores a 0 in every metric for the 'spiritual'
label, but scores better in other global metrics (averaged over all labels).

### Comment on Performance

The reason for the performance is most likely the lack of problem-specific feature
extraction. The feature extractors that both classifiers use in the current
implementation, in fact, is simply that both use the default feature extractor
provided with TextBlob [^features].

This feature extractor simply looks at all the words contained within the train set,
and encodes features as the occurrence of the word within the document.
This means that when the classifier is asked to classify a document it will look
simply at what words in the vocabulary seen at training time occur in the it.
The purpose of having a development set is exactly to improve our feature
extraction, but the timeline and priorities of the current project unfortunately
 left no time to work on this aspect of the system.

Secondly, given that the data is split in the exact same way for evaluation
of the classifier, the failure to correctly label any of the "spiritual" category
data points may have to do with idiosyncrasies with the data split, such as no
data points of that category occurring in the test set. This has partly to do with
the small amount of data available, but could be alleviated using more advanced
evaluation techniques, such as k-fold validation (Bird et al, 2015, Chapter 6
section 3.5; Sebastiani, 2002, pp.9-10).

Furthermore, see the discussion of sequence classifier for what is perhaps a more
interesting direction than simply improving the performance single-label classifers
(#REFERENCE).

[^features]: See     
<http://textblob.readthedocs.io/en/dev/_modules/textblob/classifiers.html#basic_extractor>

## Word2Vec Synonym Generation Evaluation

The model used in the implementation is a publically available model that has
been trained over a very large data set of Google News data[^word2vec].  
As mentioned in Chapter 2 and 4, the implementation here for synonym extraction
was not used in the system delivered by the team because of poor performance.

[^word2vec]: The publically available model used is not provided with the system, but obtainable from:
  <https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit>

As it is possible to see from the integration tests
the cosin distance does not always appear to capture semantic synonymity (Mikolov, 2015; McCormick, 2016b
  ). For example, while perfomance may be judged adequate with
respect to words like "dad" or "friend", there are cases when it can only associate
the word with either misspellings or completely irrelevant terms (such as names
  of reporters). This is maybe interesting linguistically, but does not help
with generating synonyms.

~~~ python
#from tests/tests_integration/test_synonym/test_synonymmodelfactory.py
{
    "dad":["Dad","father","grandpa","daddy","mom","stepdad","son","granddad",
            "uncle","brother"]
    ,"hopeless":["utterly_hopeless","forlorn","hopelessly","miserable",
                "pathetic","pitiful","helpless","seemingly_hopeless","bleak",
                "futile"]
    ,"education":["eduction","eduation","LISA_MICHALS_covers",
                    "Matt_Krupnick_covers","educational","educa_tion",
                    "edu_cation","educations","professionals_CEC_SmartBrief",
                    "curriculum"]}
~~~

As mentioned in the discussion, the implementation of this concrete class was
done with the help of the Gensim library (Rehurek, 2014; Rehurek and Sojka, 2010;
  McCormick, 2016a).  The cause
  for the poor performance with respect to the task at hand is perhaps to be found
  with the dimension of the vectors the model used was trained with (as a training parameter, see
  Ellenberg, 2015).
  The reader is redirected to the discussion of alternatives
in Chapter 2.

In general for the project going forward,
as long as RiveScript or similar rule-driven chatbot frameworks are used,
it may well be possible to similarly generate data for use in similar ways,
prior exploration of other alternatives to the particular model used in this implementation.


# Advanced Research into Categorization

So far, we have looked at how classifiers would be used to give labels to distinct
documents, however the documents we are interested in classifying
are not independent from each other: a conversation is a rich in context, and no
feature of the approach outlined so far even considers this aspect of the problem.

One thing we could do to improve the classifier is to provide a level of uncertainty,
where user inputs may fail to be assigned any of the predefined labels. This not an
attempt to account for essentially neutral inputs such as "Yes" or "I don't know",
that do not contribute to establishing the topic of the conversation. This is rather
a way to try and account for indeterminateness of the topic at hand.

If a patient were to mention an issue that scores a reasonable confidence level from
the classifier in both emotional and family categories, but both scores are below
a certain threshold, then perhaps the best thing for the classifier is to not assign
a label to the input. This accounts for the inherent "fuzziness" of certain topics of discussion.

Something else we could do is consider conversation topics as the states in a
finite state machine, and account for varying probabilities to move into different
topics, given the current state. Emotional issues may, for example, seem closer to family
issues than physical issues. However, this seems quite an ambitious architecture to
impose over a simple classifier, which leads us onto the next topic.

## Sequence Classifiers

A sequence classifier is similar to a single or multi-lable classifier, except
it considers each data point as a member of a sequence: the data points are assigned
labels not independently of each other but as part of one sequence
(Bird et al, 2015, Chapter 6 sections 1.6-1.7; Jurafsky and Martin, 2014, p.1).
This type of classifier intuitively better suits the task of classifying sequences
of utterances in a conversation, than a classifier that only decides a label for
the input considered in isolation.

A Markov chain is a weighted finite state automaton (a directed weighted graph
with a finite number of nodes), where the weight on all transitions from any state
represent the probability of moving to the state the transition points to (Jurafsky and Martin, 2014, p.2).
[#PICTUREWOULDBENICE?]

More formally:
A set of states:
$Q = \{q_1, q_2, ..., q_n\}$
A transition probability matrix:
$A = \{ a_{01}, a_{02}, ..., a_{n1}, ...,  a_{nn}\}$
And a start and an end (accepting) state: $q_0, q_F$

A Markov chain may already be sufficient for us to do some useful modelling. In particular,
we may construct a simple (ergodic, fully connected) directed weighted graph
(with appropriate constraints on the weights to represent a probability distribution
from each state) that connects our five states together
represeting the macro categories of concerns in the CC. Having manually set probabilities
for state transitions, we would use our simple classifier to decide the category
for the next user input in isolation, then confront this result against the likelihood
of a transition in that direction, before finally deciding the label.

We could take the problem one step further: a Hidden Markov Model (HMM) is a generalization
of a Markov chain that distinguishes between observed and hidden events. For example,
in a part of speech tagging task, the observed is the surface level sequence of words,
the hidden is the sequence of tags describing the syntactic structure of the
utterance. This distiction would allow, in our case, to distinguish between a sequence
of natural language user inputs, and the sequence of conversational topics "hidden"
within.

Formally, to the properties of a Markov chain outlined above we add:
A sequence of non-hidden observations:
$O = \langle o_1, o_2, ... o_m\rangle$
A function describing the probability of observation $o_j$ being generated (called probability of emission) from state $i$:
$b_i : O \mapsto [0, 1]$

The tasks that Jurafsky and Martin identify for such models are three (ibid, p.6):

- Likelihood: given a model M and a sequence of observations $O$, determine $P(O|M)$
- Decoding: given a model M and a sequence of observations $O$, determine the "best" corresponding
sequence of states $Q$
- Training: given an observation sequence $O$ and a set (not a sequence) of states $Q$,
train a model M

For our problem: at any point during the conversation, we are really interested
 in determining the probability distribution of the *reachable states* given the sequence of observations
and the corresponding sequence of states so far identified, in order to determine
what the best *next state* is. Given the next observation,
we compute the probability of emission of that observation from all the plausible states
reachable from the current state. We then may want to do a harmonized average of
these probabilities and the known state transition probabilites, in order to decide
the next state (the next label for the user input in our case).

The problem that still needs to be addressed is how we determine the emission probability
distribution. The user input will likely vary from session to session: no two sequences
of user inputs, representing the half of the conversation the user contributes, are
going to be alike.

It may be possible to reduce each user input to a set of semantically salient terms,
terms that are associated with one or another topic. For example, mention of
proper names or nouns denoting family members (such as mother, aunt etc) may be
indicative that the topic is family. In contrast, terms associated with emotional
problems may be indicative of the topic being emotional issues. In a way, we are returning
back to a simple classifier to decide what the observation to be then fed to the
sequence classifier should be.

## The limits of the NLTK

The implementation produced by this project does not make use of sequence classification, but
instead uses simple classifiers that consider each user input in isolation. There are two
reasons for this.

The first is that, as may have become evident through the preceeding section, the
problem is complex enough to warrant exploration in a separate dissertation. And
given that the focus of the current project is laying down the fundamental architecture
of a chatbot, to dedicate more resources to this topic would
require neglecting other, perhaps more important aspects of the problem to be solved.

The second is that the NLTK does not provide implementations of any sequence
classifier (at the time of writing). Looking through the public open source
repository it is possible to see that an interface sequence classifiers would be
expected to conform to within the NTLK package has been written, but is commented out
in the code, and no implementation
of the interface can be found[^nltksource].

There are open source implementations of Hidden Markov Models available, but
work would be required in order to either fit these to the NTLK system, or to
incorporate them within the current project. See for example, "pomegranate"
(Schreiber et al, 2016)[^pomegranate].

[^nltksource]: This link should reference the latest commit to the NLTK project as of 21/08/16:
 <https://github.com/nltk/nltk/blob/991f2cd1e31f7c1ad144589ab4d2c76bee05aa7b/nltk/classify/api.py>.

[^pomegranate]: Here: <https://github.com/jmschrei/pomegranate#hidden-markov-models>

# Code Listing

Partial code listings follow. The full source code can be obtained following
the instructions in Appendix A.

# References
