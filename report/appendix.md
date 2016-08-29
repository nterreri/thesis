\appendix

# System Manual

As stated before, the scope of this report is limited to the core chatbot system
which interfaces (in the larger team effor) with a webserver for delivery to
a user. For this reason, no user manual is provided.

## Installation

The full project source code (including the code of the other team members)
is available at: <https://github.com/andreallorerung/peach-chatbot-alpha>.
The author's contribution to the project can be found under the
'/FlaskWebProject/chatbot/' directory therein
 (<https://github.com/andreallorerung/peach-chatbot-alpha/tree/master/FlaskWebProject/chatbot>).
This can also be found at <https://github.com/nterreri/peach-bot>.

In order to obtain the code it is possible to download the repository as a zip
file from the relevant <github.com> webpage, or use the git software to clone into the
repository (*$ git clone https://github.com/andreallorerung/peach-chatbot-alpha*). The
categorizer package is provided as a git submodule to the chatbot package.
Cloning or downloading the repository will not automatically download the submodule
content. To download all submodules when cloning, use
*$ git clone --recursive https://github.com/andreallorerung/peach-chatbot-alpha*.
It is not possible to do the same if downloading the repository as a zip file,
which means it is necessary to navigate manually to the categorizer repository
(<https://github.com/nterreri/peach-categorize>) before downloading it.

When wanting to download the submodule after having cloned into the repository
non-recursively, use *$ git submodule update --init --recursive* to download
the submodule.

The project is meant to be compiled through a Python 2.7 interpreter, no support
is provided for Python 3.x. This is primarily because of the preference of memebers
of PEACH. The project has been only compiled and tested using the default C
Python compiler which should come by default with any official Python distribution.

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

### The *botinterface* package

The *botinterface/* package exposes the interface class for the core chatbot
component in the *botinterface/bot_interface.py*, and provides an implementation
in *botinterface/bot_rivescript.py*. In order to depend only on the API, it is
possible to ask the *botinterface/bot_builder.py* package to construct a concrete
subtype of the interface.

This last module exposes a *build()* method that will return a default instance
of the bot, which will assume the RiveScript brain files are located within
a *brain/* directory immediately accessible. Should that not work, the same
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
the chatbot component.

As mentioned before, the conversation logging facilities provided by the
*messagelog* package are not automatically used by the chatbot instance, instead
something like code listing (#SOMENUMBER) is necessary.

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
a lot of pages to be swapped in and out of virtual memory (on modern operating
systems) as the model retrieves similar words based on cosin distances. Sample
usage of this module (adapted from */tests/tests_integration/test_synonym/test_synonymmodelfactory.py*):

~~~ python
synonymsFor = dict()
for word in WORDS_TO_GET_SYNONYMS_FOR:
      synonymsExtracted = extractor.extractSynonyms(word)

      synonymsFor[word] = synonymsExtracted
~~~

This will return a list of words which the model believes are synonyms of the
argument to the *extractSynonyms()* method. As mentioned in Chapter 5, this
particular implementation was underperforming, but in the future may provide
the blueprint to other better performing implementations.

## Running py.test

The tests included are meant to be run using py.test (Krekel, 2016).
In order to allow the required project packages to be imported, this should be
run as follows, the directory or test file to run can be specified as an
argument to the Python CLI interpreter:

~~~
python -m py.test tests/
~~~

Running this command will run all the tests in the package, which on a fairly
powerful machine can take up to 5 minutes, but can potentially make a slower
machine hang as for an unreasonable amount of time. This is chiefly due to the
synonym generation tests which require a very large amount of RAM to run efficiently.
It takes around 3-4 minutes on an 8-core desktop with 8GBs and the amount
of memory required to run
it *without* thrashing happening as pages are swapped in and out of virtual memory
 is higher than 8GBs.

It is good to notice at this point the difference in runtime between the decoupled unit tests
and the integration tests in this case, running all unit tests for the project takes
mere seconds (as it should).

The unit tests, in fact, are designed for quick execution (they are meant
to be ran many times an hour during TDD; see F.I.R.S.T. Martin, 2009, p.132).
Running these tests should take less than a minute.
Finally, if the pytest-cov package has been installed (Schlaich, 2016) then it
is possible to get reports for code coverage about individual subpackages
by providing the desired subpackage name as an argument in the following manner:

~~~
python -m py.test --cov=botinterface tests/tests_unit/
~~~

This will provide an executable statement test coverage report. See (#WHICHLETTER)
below for results collected.

The reader is redirected to Chapter 5 and Appendix (#WHICHLETTER) for more
information about the tests.

# Tests Results

Unit tests coverage for the *botinterface* package:

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
-----------------------------------------------------------
TOTAL                                     139     12    91%
~~~

Unit tests coverage for the *categorize* package[^pickle]:

~~~
---------- coverage: platform linux2, python 2.7.9-final-0 -----------
Name                                   Stmts   Miss  Cover
----------------------------------------------------------
categorize/__init__.py                     0      0   100%
categorize/classifierDeserializer.py       5      0   100%
categorize/classifierSerializer.py         4      0   100%
categorize/dataset_reading.py             12      4    67%
categorize/dataset_splitting.py           10      0   100%
categorize/develop.py                     10     10     0%
categorize/evaluation.py                  46      2    96%
categorize/training.py                     4      0   100%
----------------------------------------------------------
TOTAL                                     91     16    82%
~~~

Unit tests coverage for the *concerns* package:

~~~
---------- coverage: platform cygwin, python 2.7.10-final-0 ----------
Name                                      Stmts   Miss  Cover
-------------------------------------------------------------
concerns/__init__.py                          0      0   100%
concerns/concern.py                          10      0   100%
concerns/concern_factory.py                  13      0   100%
concerns/drive_conversation.py               54      0   100%
concerns/drive_conversation_abstract.py      11      5    55%
concerns/equivalence.py                      32      1    97%
concerns/rivescriptmacros.py                 24      3    88%
concerns/topics.py                           14      0   100%
-------------------------------------------------------------
TOTAL                                       158      9    94%
~~~

Unit tests coverage for the *messagelog* package:

~~~
---------- coverage: platform cygwin, python 2.7.10-final-0 ----------
Name                                 Stmts   Miss  Cover
--------------------------------------------------------
messagelog/__init__.py                   0      0   100%
messagelog/conversation.py               6      0   100%
messagelog/conversation_logging.py      21      0   100%
messagelog/message.py                    8      0   100%
--------------------------------------------------------
TOTAL                                   35      0   100%
~~~

Unit tests coverage for the *postprocess* package:

~~~
---------- coverage: platform cygwin, python 2.7.10-final-0 ----------
Name                                      Stmts   Miss  Cover
-------------------------------------------------------------
postprocess/__init__.py                       0      0   100%
postprocess/keyword_extractor.py              5      2    60%
postprocess/keyword_extractor_single.py      30      0   100%
postprocess/message_decorator.py              5      2    60%
postprocess/message_decorator_single.py      29      0   100%
postprocess/postprocessor_builder.py          5      0   100%
postprocess/search_adapter.py                 5      2    60%
-------------------------------------------------------------
TOTAL                                        79      6    92%
~~~

Unit tests coverage for the *preprocess* package:

~~~
---------- coverage: platform cygwin, python 2.7.10-final-0 ----------
Name                                      Stmts   Miss  Cover
-------------------------------------------------------------
preprocess/__init__.py                        0      0   100%
preprocess/preprocessor_builder.py            6      0   100%
preprocess/stemmer_factory.py                 3      0   100%
preprocess/stemming.py                        5      2    60%
preprocess/stemming_lancaster.py              7      1    86%
preprocess/stopword_remover_nltk.py           6      0   100%
preprocess/stopwords_remover.py              17      1    94%
preprocess/stopwords_remover_lenient.py       5      0   100%
preprocess/tokenizer.py                       5      2    60%
preprocess/tokenizer_simple.py                7      1    86%
-------------------------------------------------------------
TOTAL                                        61      7    89%
~~~

Unit tests coverage for the *synonym* package:

~~~
---------- coverage: platform cygwin, python 2.7.10-final-0 ----------
Name                                   Stmts   Miss  Cover
----------------------------------------------------------
synonym/__init__.py                        0      0   100%
synonym/rivescript_array.py                9      0   100%
synonym/rivescript_writer.py               7      7     0%
synonym/store_synonyms.py                  4      4     0%
synonym/synonym_extractor.py               5      2    60%
synonym/synonym_extractor_factory.py      13      5    62%
synonym/synonym_word2vecextractor.py      17      0   100%
----------------------------------------------------------
TOTAL                                     55     18    67%
~~~

Total:

- 527 executable statements
- 52 missed statements
- 88.83% averaged percentage

Note that there are several abstract classes that are also counted in the total
executable statements even though they should not, for example *MessageProcessor*:

~~~ python
class MessageProcessor(object):
    '''Interface a message content processor is expected to implement'''
    def __init__(self):
        raise NotImplementedError("MessageProcessor is an interface")

    def process(self, sentence):
        '''To preprocess the sentence (content of a message)'''
        raise NotImplementedError("MessageProcessor is an interface")

~~~

The two "raise" statements are counted when they should not be, similarly with
many other classes.

The tests also test the RiveScript brain data, and possible conversation paths
within it, specifically it is the integration tests for the *botinterface* package
that test "conversation cases" against the chatbot, for example:

~~~ python
def test_work():
    resetpractical()

    # perform:
    messages = ["I am underperforming at work because of stress",
    "Something about work ...", "I am afraid to lose my job"]

    for msg in messages[:]:
        reply = bot.reply(Message(USERID, msg))
        # test:
        found = False
        good_replies = ["Are you afraid for your job security?",
        "Would you say this is also a money concern?",
        "How has your condition been affecting your work?"]

        for good_reply in good_replies:
            if good_reply == reply:
                found = True
                break

        assert found, reply
~~~

This test first resets the conversation to the "practical" topic in the RiveScript
brain, then sends topic pertinent user chat messages to the bot for processing.
The both reply is tested at each message against a set of pertinent replies (
  which represents all of the replies that have been coded into the chatbot
  brain for that particular type of topic).
This and other tests like it are run to verify the behaviour of the chatbot at
the brain level (the messages are piped through the BotRivescript object).
Furthermore, other tests verify that the chatbot brain changes its state based
on the topics that have been discussed throughout the conversation, and other
criteria (such as whether it is appropriate to make a query through the search
  enginge) are also run. All such tests can be found between the
*tests/tests_integration/test_botinterface* and *tests/tests_integration/test_concerns*
folders.

Finally, the RiveScript Python macro code within *FlaskWebProject/chatbot/brain/python.rive*
is not considered and not covered by tests and is in fact impossible to test in isolation. This is because
of the fact that the RiveScript object instance "rs" accessible within these
macros cannot be called from inside separate and isolated methods (as it was the
author's original intention) and will raise an error if it is passed as a variable
to an externally defined unit of Pyhon code. This aspect of RiveScript is poorly documented, and
another reason to move away from the framework in the future.

[^pickle]: You may notice the different platform running the tests for the
categorizer (linux) this is due to an issue with the pickle package on Windows,
see: <https://github.com/DataTeaser/textteaser/issues/3>

# Categorizer Evaluation

The single-label categorizers used in the project where the two that are adapted
from the nltk into the TextBlob package. A Naive Bayes classifier and
a Maximum Entropy classifier.

The results of running the first against a test set (which is 10% the size of
  the complete dataset of 222 data points, around 22 test cases) is disappointing:

~~~
Accuracy: 0.36

Recall for 'phys': 0.67
Precision for 'phys': 0.33
F for 'phys': 0.44
Recall for 'pract': 0.0
Precision for 'pract': 0.0
F for 'pract': 0
Recall for 'family': 0.4
Precision for 'family': 1.0
F for 'family': 0.57
Recall for 'emotion': 0.4
Precision for 'emotion': 0.4
F for 'emotion': 0.4
Recall for 'spiritual': 0.0
Precision for 'spiritual': 0.0
F for 'spiritual': 0

Macro averaged recall: 0.29
Macro averaged precision: 0.35
Micro averaged recall: 0.36
Micro averaged precision: 0.36
~~~

All metrics for the 'pract' and 'spiritual' labels are 0 which means the classifier
systematically gets them wrong over the data set. The micro and macro averages
of precision and recall for all labels score around 33%.

The MaxEntClassifier, instead, scores over the same data split (10% test, 10% dev,
  80% train) when made to iterate over the dev set for 10 iterations:

~~~
Accuracy: 0.55

Recall for 'phys': 0.83             |Macro averaged recall: 0.4467
Precision for 'phys': 0.56          |Macro averaged precision: 0.4911
F for 'phys': 0.67                  |Micro averaged recall: 0.5454
Recall for 'pract': 0.4             |Micro averaged precision: 0.5454
Precision for 'pract': 0.4
F for 'pract': 0.4
Recall for 'family': 0.4
Precision for 'family': 1.0
F for 'family': 0.57
Recall for 'emotion': 0.6
Precision for 'emotion': 0.5
F for 'emotion': 0.55
Recall for 'spiritual': 0.0
Precision for 'spiritual': None
F for 'spiritual': None
~~~
Like the Naive Bayes classifier, it scores a 0 in every metric for the 'spiritual'
label, but scores better in other global metrics (averaged over all labels).

## Comment on Performance

The reason for the performance is most likely the lack of problem-specific feature
extraction. The feature extractors that both classifiers use in the current
implementation, in fact, is simply that both use the default feature extractor
provided with TextBlob (see <http://textblob.readthedocs.io/en/dev/_modules/textblob/classifiers.html#basic_extractor>).

This feature extractor simply looks at all the words contained within the train set,
and encodes features as the occurrence of the word within the document.
This means that when the classifier is asked to classify a document it will look
simply at what words in the vocabulary seen at training time occur in the it.
The purpose of having a development set is exactly to improve our feature
extraction, but the timeline of the current project unfortunately left no time
to work on this aspect of the system.

Furthermore, see the discussion of sequence classifier for what is perhaps a more
interesting direction than simply improving the performance single-label classifers
(#REFERENCE).

# Code Listing

Partial code listings follow. The entire source code can be obtained following
the instructions in Appendix A.

# References
