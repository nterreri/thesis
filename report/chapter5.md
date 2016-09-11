# System Testing and Evaluation

>> *"The act of writing a unit test is more an act of design than of verification."*[^martin]

[^martin]: Martin, 2003, Chapter 4.

This chapter describes the process followed to produce the suite of tests that
verify the functioning of the system at the level of core logic and the higher
level system behaviour. This chapter also describes the process of
classifier evaluation provided by the classification package.
The full test coverage results and machine learning evaluation listings are found in
Appendices B and C.

## Testing Strategy

Given the topic of the project, the testing strategy adopted focuses on verifying
the behaviour of the individual units of code, the integration of units together
as well as performance intensive operations (Gensim) or units that perform heavy
IO (RiveScript).
This is accomplished through systematic testing
with a test-first approach: the test defining the specification for the next
system increment is constructed before the unit that will have to pass it
(Beck and Andres, 2004, pp.50-51, 97-102; McConnell, 2004, pp.499-).

The integration tests also verify the behaviour of the chatbot brain, where
messages are sent to the bot brain and the test fails if the reply returned
does not fall within a predefined range of acceptable replies. For example:

~~~ python
def test_work():
    resetpractical()
    # perform:
    messages = ["I am underperforming at work because of stress"
    , "Something about work ...", "I am afraid to lose my job"]

    for msg in messages[:]:
        reply = bot.reply(Message(USERID, msg))
        # test:
        found = False
        good_replies = ["Are you afraid for your job security?"
        , "Would you say this is also a money concern?"
        , "How has your condition been affecting your work?"]

        for good_reply in good_replies:
            if good_reply == reply:
                found = True
                break
        assert found, reply
~~~
This test first resets the conversation to the "practical" topic in the RiveScript
brain, then sends topic-pertinent user chat messages to the bot for processing.
The both reply is tested at each message against a set of pertinent replies (which
    represent the replies that have been coded into the chatbot
  brain for that particular type of topic). This and other tests like it are
  run to verify the behaviour of the chatbot at
the brain level.
Furthermore, other tests verify that the chatbot brain changes its state based
on the topics that have been discussed throughout the conversation, and other
criteria (such as whether it is appropriate to make a query through the search
  engine). All units and integration tests were made to pass.

One other important form of testing is acceptance testing. The lack of this sort
of testing from the project is due to the tight schedule of project delivery and
the fact that while there were initial hopes of getting cancer patients to try
out the software these were later dismissed as concerns were raised about
getting access to the general public via the UCL Hospital.

## Test Driven Development

The test-first strategy for testing the core logic of the project is TDD (Martin, 2009, pp.122-123)[^TDD].
There are three main benefits associated with this practice. The first is that
it encourages to take on the perspective of the caller of the code, before any
code has been written which makes the code easy to call and use to the programmer
who is going to consume it. Secondly, it encourages a loosely coupled design.
Testing the BotRivescript class in isolation, for example, requires that its
associated objects be replaced with mocks providing predictable behaviour, so
that the logic of the class can be scrutinized without external interferences.

Third, it leads to self-documenting code. The tests
describe how to use the system and as long as the tests
are continuously made to pass, they will never be out of date as comments and
Docstrings are always at risk of becoming[^notall].
For example, the following function of the messagelog package
integration test demonstrates how to use both the chatbot and message
logging facilities provided by the present package in a way that is technically
accurate and intuitive
enough that programmers should be able to understand how to use
it without further guidance (*sendUserMessageAndLog(userid, message)* in code listing E.8 Appendix E).

[^TDD]: See also Hansson (2014) and Fowler et al (2014).

[^notall]: Not all tests were written strictly adhering to the rules of TDD because the author was learning
the discipline while working on the project.
As a result, the later a unit was designed, the better the tests for it were.

The employement of TDD enabled the tests for the system to achieve 95% code coverage.
See Appendix B for test coverage listings and more details.

## Categorization Evaluation Strategy

A classifier is evaluated by looking at metrics of its performance.
A very simple metric is accuracy: the number of correctly labelled
data points over the total number of test cases (Bird et al, 2014, Section 3.2).

The way the categorization package provides an evaluation of the classifier is
through the *ClassifierEvaluator* class. The interface of this class allows access
to accuracy, precision, recall and F metrics for the given classifier over a
"gold" standard data set (Perkins, 2010; Sebastiani, 2002, pp.32-37). Perhaps the most useful of these is
the F measure, as Sebastiani (ibid, p.34) warns that accuracy tends to be
maximized by the "trivial rejector" or the classifier that tends to assign no label
to all data points.

It is important to note, however, that the precision and recall the evaluation
module provides for a classifier are individual to the particular
label being tested for, and either macro- or micro-averaging should be used
to extract global classifier effectiveness metrics (Sebastiani, ibid p.33;
Yang and Liu, 1999, p.43).

The type of data we want to produce a label for is a short chat
message. The label has to be in the specified range of categories (Section 2.1.1), in order to
better inform the chatbot with the capacity for generalization of a machine learning
approach. There is no corpus of pairs of message - label of the relevant kind.
Therefore, the author created a survey which Dr Ramachandran asked the team to
complete[^survey].
This resulted in 222 data points.

[^survey]: The survey can be found here: <https://goo.gl/forms/ylkI50XckV9yvCCm2>.

The survey is a series of questions the chatbot may ask, grouped by topic. The
label for each answer gathered is inferred from the implicit topic of the quesiton.
For example: a question about physical pain belongs to the physical concerns label.
Since all of the data extracted via the questionnaire
has a balanced number of expected labels for categories, there is no
concern about the training data being skewed with low positive cases for any one
category, therefore micro-averaging is probably the best overall measure, although both are
included in the evaluation results (Sebastiani, 2002, p.33).

The performance of the categorizer implementations tried
are disappointing, primarily because of lack of problem-specific feature selection
and possibly idiosyncrasies in the test set. See Appendix C for full results listings
and further discussion.
