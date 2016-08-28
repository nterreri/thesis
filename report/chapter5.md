# System Testing and Evaluation

> "The act of writing a unit test is more an act of design than of verification."

**Martin, 2003**

This chapter describes the process followed to produce the suite of tests that
verify the functioning of the system at the level of core logic and the brain
matchers, redirects and coditionals. This chapter also describes the process of
classifier evaluation provided by the classification package.

## Test Driven Development

TDD is described as a discipline prescribing adherence to three rules whenever
writing code (Martin, 2009, pp.122-123):

1. Do not write any production code before you have a failing unit test for that code.
2. Do not write more of a test than is sufficient for it to fail (not compiling counts as failing).
3. Do not write more production code than is sufficient to pass the test.

There are three main benefits associated with this practice. The first is that
it encourages to take on the perspective of the caller of the code, before any
code has been written. This makes the code easy to call and use to the programmer
who is going to consume it.

Secondly, it encourages a loosely coupled design.
This is because a *unit* test is meant to test a unit of code (be it a procedure,
a subroutine, function or class) in isolation; it is the most focused type of
test possible. This means that any external
dependencies, interactions and collaborations ought to be mocked for the purpose of the unit test.
Testing the BotRivescript class in isolation, for example, requires that its
associated objects be replaced with mocks providing very predictable behaviour, so
that the logic of the class can be scrutinized without any external interference.

Third, the source code always tells the truth. Failing all other methods of
documentation, from diagrams to Python Docstrings or Javadoc, the source code
is the ultimate documentation of a software project. Unit and integration tests
provide the documentation that describes how to use the system and any and all
failure modes or edge cases for that system. Furthermore, as long as the tests
are continuously made to pass, they will never be out of date as comments and
Javadoc are always at risk of becoming.

For example, the \_sendUserMessageAndLog() function of the messagelog package
integration test demonstrates the way to use both the chatbot and message
logging facilities provided by the present package in a way that is technically
accurate to the point that it could be compiled to production, and intuitive
enough that programmers looking at it should be able to understand how to use
those facilities without further guidance[^TDD].

``` python
def _sendUserMessageAndLog(userid, message):
    ConversationLogger.logUserMessage(message)
    reply = bot.reply(message)
    assert reply is not None
    ConversationLogger.logSystemReplyForUser(reply, userid)
    return reply
```

[^TDD]: TDD is not free of controversy, see Hansson (2014) and Fowler et al (2014).

## The Project Tests

The truth about the 120 unit tests in the present project is that the author
was learning what TDD was during development, hence not all tests were written
according to the three "rules" of TDD. A test-first approach to software
development, however, has been taken from the start (Beck and Andres, 2004, pp.50-51, 97-102).
As a result, the later a unit was designed, the better the tests for it were.

Integration tests were produced to test the interaction of different units. These
were not always pairwise (two units at a time), but were carried out depending on
the interactions the author was concerned with documenting and verifying (McConnell, 2004, p.499).
The 66 integration tests include numerous tests of the chatbot brain, piped throught
the complete BotRivescript façade.

One other important form of testing is acceptance testing. The lack of this sort
of testing from the project is due to the tight schedule of project delivery and
the fact that while there were initial hopes of getting cancer patients to try
out the software these were later dismissed as concerns were raised about
getting access to the general public via the UCL Hospital.

One final note about the tests: the pre-trained word2vec model used in the integration
tests which is necessary for the Word2VecSynonymExtractor implementation to work
(not provided with the system, but obtainable from: https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit)
is very demanding in terms of main memory. Running the tests included in the
source code takes around 3-4 minutes on an 8-core desktop with 8GBs and the amount
of memory required to run
it without thrashing is higher than 8GBs. Be wary when running these tests, as
they may effectively impair slower machines for an unreasonable amount of time.
It is good to notice at this point the difference in runtime between the decoupled unit tests
and the integration tests in this case, running all unit tests for the project takes
mere seconds (as it should).

(#ADD PYTEST COVERAGE REPORTS)

## Categorization Evaluation

A classifier is evaluated by looking at certain metrics of its perfomance during
testing. A very simple metric is accuracy: the number of correctly labelled
data points against the expected label (Bird et al, 2015, Section 3.2).
It may also be useful to build a "confusion matrix" (ibid, Section 3.4).

In this matrix, the diagonal represents the percentage of correctly labelled samples,
the column for each category for cells not lying on the diagonal represent the percentage
of times the labelled indicated by the column was mistaken for the labelled indicated in the row.
A simple graphical representation of such a matrix can be obtained via the *confusionmatrix*
module of the *metrics* subpackage of the NLTK: http://www.nltk.org/_modules/nltk/metrics/confusionmatrix.html.

The way the categorization package provides an evaluation of the classifier is
through the ClassifierEvaluator class. This class' interface allows access
to accuracy, precision, recall and F metrics for the given classifier over a
"gold" standard data set (Perkins, 2010; Sebastiani, 2002, pp.32-37). Perhaps the most useful of these is
the F measure, as Sebastiani (ibid, p.34) warns that accuracy tends to be
maximized by the "trivial rejector" or the classifier that tends to assign no label
to all data points.

It is important to note, however, that the precision and recall the evaluation
module provides for a multi-lable classifier are individual to the particular
label being tested for, and either macro- or micro-averaging should be used
to extract global classifier effectivness metrics (Sebastiani, ibid p.33;
Yang and Liu, 1999, p.43).

Because of the fact that all of the data extracted via the questionnaire (see
Chapter 2) has a balanced number of expected labels for categories, we are not
concerned about the training data being skewed with low positive cases for any one
category, micro-averaging was chosen as the balance of effectivness for all
categories, with the F measure being the final combined evaluation measure.

(#PENDING IMPLEMENTATION OF MICROMACROAVG)
