# Background Research

This chapter details the literature review for the project: the eHNA software,
the background reading, and the tools and frameworks selection process.

## The electronic Health Needs Assessment questionnaire

The primary reference for the software to be built by the team
is Macmillan's eHNA. Macmillan Cancer Support developed the eHNA for the purpose of extending the
range of cancer patients in the UK covered by individual care plans, made to address
the individual's very personal and unique concerns they incurred into in relation
to their condition. These concerns are gathered through variants
of an electronic questionnaire offered by Macmillan to selected trial sites.
Paper versions and variants of the questionnaire
existed before the introduction of the eHNA in 2010, and have been in use since before then
(Watson, 2014; Brittle, 2014; NCAT, 2011).

The electronic questionnaire is designed to be carried out on site mostly through
haptic devices (such as tablets), just ahead of meeting the clinician that will help draft a care plan for the patient. There
is the option to complete the questionnaire remotely, although the adoption of
this alternative is made difficult by the work habits of key personnel, who are
used to providing a device to the patient in person and ask them to carry out the
questionnaire while at the clinic.
The patient uses a touch interface to navigate through various pages
selecting concern categories from a predefined list. There are several versions
of questionnaires available, modelled after the various paper versions, depending on which
one the clinic previously used.
The front end of the system is implemented as web-app.
Access to the assessment is restricted to scheduled appointments that clinics set up
for individual patients, either via delivering the questionnaire on the clinic site,
or, if the questionnaire is carried out remotely, via use of a one-time 6 digit PIN
number, alongside the patient's name and date of birth.

### The Concerns Checklist {#ConcernsChecklist}
Given the variety of different versions of the questionnaire, the team was advised
by the client to focus on the one that is most commonly used: the Concerns Checklist (NCAT, 2011).

In this version of the questionnaire, the patient selects their concerns from a
range of more than 50 individual issues, each falling into one of 10 categories,
and selects a score for it in a range from 1 to 10.
Each category may itself be a subcategory of the following major topics (see Appendix A.3 for
a full list):

- Physical concerns
- Practical concerns
- Family concerns
- Emotional concerns
- Spiritual concerns

## Patient Data for Research in the UK
As mentioned in Section 1.2, handling confidential
patient data poses particular challenges to eHealth related projects. Just before
the start of the project, the
whole PEACH team underwent training about handling patient data and the relevant
legislation in the UK. These affected
design and implementation decisions.

Generally speaking, authorization is required before any information provided
by the patient can be used in any way except the specific purpose of their healthcare
(Data Protection Act 1998).
It may be possible to make use of third party services provided
the data has been fully anonymized and cannot be linked back to the patient.
Preferably, a special agreement (such as a Data Transfer Agreement) should be
brokered to ensure both parties understand the legal and ethical implications
of sharing even anonymized data (in such cases, the duty of confidence, UK common law,
 does not extend over to the third party).
Note however, that it is sometimes difficult to ensure that data has been anonymized, even
by removing all information considered personal under UK law: for example, if
a person happens to have a rare disease, or information about the geographic
location of the patient can be retrieved from the data being shared with the third
party.

For this reason, the implementation of the current project does not share any of
the data extracted from user input with external third parties although the
architecture will allow for such a choice to be made in future iterations of
the chatbot project.

## The Chatbot

The very high level general architecture of a chatbot system is normally described in terms of
(Bird et al, 2014, Chapter 1 section 5.5; Jurafsky and Martin, 2009, pp.857-867):

1. A natural language understanding layer
2. A dialogue and or task manager
3. A natural language generation layer

The first may be simply implemented through hand-written finite state or context free
grammars. The system takes in the user input and attempts to parse it according
to predefined grammar rules in order to extract the relevant information
(Jurafsky and Martin, ibid, p.859).

The simplest way to address number 3 is to have a set of hand-written natural language
templates that are filled in with relevant information before being output to
the user.
A way to avoid having to hand-write the grammar rules is by making use of a probabilistic
parser that also seeks to fill-in slots of information required by the system to carry
out the task (ibid. p.860).
Finally, the dialogue or task manager would be what models the information
required of the user for the completion of the task, and manages the turn taking
and the grammar rules to match for the user inputs and the templates to use in
output.

This type of rudimentary system can be contrasted with more complex systems
that include modelling of the conversational context and understanding and generation
modules that can operate with higher level abstraction than simple patterns of symbols
(like the grammar rules and templates described above), which we may want to
call "dialogue acts". These more advanced systems are sometimes referred to
"information-state" systems as opposed to "frame-based" systems that coerce
the conversation to a very specific task (Jurafsky and Martin, ibid, pp.874-875).

### The Chatbot "Brain" Market

This section aims at describing the options that were discovered while researching
ways to set up an initial chatbot. Open source projects available mostly
make use of the patter/grammar and template model described above, but there is
a notable alternative in emerging (yet immature) technology using neural networks or
similar tools to build conversational models. Finally, there are
AIaaS providers: providers of remote intelligent agents and related services
(Pandorabots, 2016; Riedel et al, 2016; Microsoft Cognitive Services, 2016), similarly to
the web hosting solutions of Amazon or Microsoft (AWS, 2016; Azure 2016).
For the reasons outlined in the above section on the particular legal issues around
the problem domain, it was deemed unfeasible to use external services that would
host the chatbot service and as a consequence receive and process patient
information (even in anonymized form).

In the architecture model proposed above, the open source resources available make
 it easy to write documents that define the patterns and templates of a "frame-based"
system, simplifying the process to build all three of the above defined layers.
We now proceed to review the various options, and motivate the choice of tool
used in this project[^chatbotSources].

[^chatbotSources]: The main sources for this section of
the report are the individual websites of the tools explored, and the forum of
<https://www.chatbots.org/ai_zone/> and related readings (Morton, 2011; Wilcox, 2011).

#### The Artificial intelligence Markup Language
AiML is a version of the Extensible Markup Language (XML) that was specifically
designed around providing a transferrable standard to define rules, patterns and grammars to
match user inputs to appropriate templates. On top of the basic patterns and
template, AiML provides ways to use wildcards or optional sub-patterns
to capture parts of the user input for processing or to repeat
back to the user by decorating the template (for examples see: Wallace, 2014).

AiML also provides ways to define topics as restrictions over the set of match-able
patterns. Entering a topic effectively means restricting the patterns that user
input can match to the ones associated with the topic. It is also possible to
set and read internal variables tied to one user, and use this state conditionally
to decide which template to use in the output; it is possible to refer back to
the previously matched input, for example to read a follow up to a yes-no question[^yesno].

[^yesno]: See <http://www.alicebot.org/aiml.html>; Wallace, 2014.

There are separate guidelines
to follow to implement a AiML reader (or "interpreter"). The set of files making up
the AiML "bot" are commonly called the chatbot "brain" and
there are a number of interpreters available for AiML in various programming languages.
There are freely available "libraries" of AiML files for others to include
into their own chatbot (Wallace, 2011)[^base].

[^base]: <https://github.com/pandorabots/rosie/>,         
<http://www.alicebot.org/downloads/sets.html>,          
<http://www.square-bear.co.uk/aiml/>.

#### RiveScript
RiveScript is an alternative standard to AiML that aims to be
as expressive and useful as AiML, but with a simpler syntax, getting rid of the XML
(Petherbridge, 2009; Petherbridge, 2012; <https://www.rivescript.com/compare/aiml>). Like AiML,
RiveScript has support for topics, remote procedure calls, conditionals, redirections
and other features[^arrayss].

[^arrayss]: See Section 2.5 for a discussion of RiveScript arrays and synonyms.

But what made RiveScript particularly attractive was not just its simpler syntax,
it was rather the fact that there is an official Python RiveScript interpreter
but (at the time of writing) there are no easily traceable
AiML interpreters implemented in Python (Petherbridge, 2016).
The best candidates are pyAIML and pyAiml-2.0
(Tomer, 2014; Iaia, 2016), neither of which has either been maintained for a long time,
or is very stable[^aimlinterpreters].

[^aimlinterpreters]: This also applies to most other open source AiML interpreters
implementations at the time of writing, leaving only a couple standing
 (ALICE A.I. Foundation, 2014; Morton and Perreau, 2014).

#### ChatScript
ChatScript is in many ways similar to RiveScript in that
it provides a very easy to read syntax (Wilcox, 2011; Wilcox, 2016b).
In ChatScript, it is possible to
define "concepts" like RiveScript "arrays". ChatScript is also integrated in WordNet: a
lexical database for the English language that primarily models synonymy and hyponymy
between English words (Fellbaum, 2005).
Like the others, ChatScript supports external procedure calls, wildcards, optional sub-patterns
and the other pattern matching features of the previous standards.

#### SuperScript
SuperScript is a fork of RiveScript with syntax elements inspired by ChatScript
(Ellis, 2016; Ellis 2014). It boasts features from all of its predecessors, including WordNet
integration, plus a complex input processing pipeline that will attempt to analyse
the user input as a question and try to provide an answer to it, in light of the
preceding conversation.

The core issue with this system is the fact that it is only made for NodeJS, in
particular, only versions 0.12 or 0.5x[^node].
This may create problems where this project is used in conjunction with NodeJS
in other applications (on the webserver for example), and while there are workarounds
to having to keep multiple versions of Node, there is the risk of making it more
and more difficult to maintain the system as Node and SuperScript evolve[^mota].

[^node]: While the author is unfamiliar
with Node, this came across as a red flag. The recommended version of NodeJS for
most users at the time of writing is 4.5.0, while the latest build version is
6.4.0 (Node Core Technical Committee and Collaborators, 2016;         
<https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V4.md#2016-08-15-version-450-argon-lts-thealphanerd>,          
 <https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V6.md#2016-08-15-version-640-current-cjihrig>).

[^mota]: See Mota, 2016 here for how to manage multiple NodeJS versions:
 <https://www.sitepoint.com/quick-tip-multiple-versions-node-nvm/>.

#### Neural-network-based conversation models
Work has been done to use various types of neural networks to produce general
purpose conversational systems. Sordoni et al (2015), for example used several
recurrent neural networks to keep account of the conversational context by modelling the information
previously processed (ibid, section 3). Shang et al (2015) use Statistical
Machine Translation techniques, treating the response generation (or "decoding")
phase of the process as a linguistic translation problem (ibid, figure 1).

Vinyals and Le (2015) have used the seq2seq (sequence-to-sequence) model that
uses a recurrent neural network to map an input sequence to an output sequence
token by token with interesting results. However, this is, as they claim, a purely
data-driven approach that relies on a significant volume of pre-existing data to train
the model. No such data exists for the specific domain of the present project, and therefore
would probably only be possible when sufficient natural language conversation data
specific to the system domain has been gathered (or generated).

#### Conclusion

AiML and competitors all seem to support the same set of basic features, but of
particular interest for the current project was the possibility to define and
control the content of topics, in order to provide only domain-relevant replies
from the system to the user. Of the four, RiveScript is the only one that explicitly
supports topic inheritance, which seemed useful with respect to creating a hierarchy
of macro and micro topics: for example, having a global scope with general purpose
commands (such as change topic) with sub-scopes like "family" and "physical" which
could be further sub-scoped to have issue-specific matchers, for example matchers that
are only relevant to respiratory problems and would not occur in the related
physical category of nausea problems, although both would share more general matchers
about physical issues (Petherbridge, 2009)[^petherbridge2009].
ChatScript also allows "enqueuing" of topics with the concept
of "pending topics" and also control of context via "rejoinders" (Wilcox, 2016a, pp.9--;
Wilcox, 2016b, pp.5--).

[^petherbridge2009]: <https://www.rivescript.com/wd/RiveScript#topic>.

Another point of interest (again, given the author's aim to explore Python) is
the open source software available for use. Given the considerations
already provided, SuperScript seemed like the least comfortable option from this
perspective (Node), with ChatScript (whose only interpreter is in C++) being second least.
This would leave RiveScript and AiML, with RiveScript's simpler but expressive
syntax and its currently maintained Python interpreter being the final deciding
factors for the current implementation.

## Natural Language Processing
As stated in Chapter 1, part of the author personal aims included learning about
NLP and leverage the author's background in computational linguistics.
Specifically, the
possibility to classify user input according to the topic being mentioned
 was identified as a useful application of NLP to the problem.
 It may be useful to add tags to user
inputs to be matched within a chatbot brain that would otherwise be unable to
generalize. This would effectively represent a hybrid model where both a machine learning
component and a "rule-based" (input-patterns) approach are
used as part of a more complex system[^experience].

[^experience]: Generalization
 is characteristic of approaches to artificial intelligence resembling
experience as a way humans acquire knowledge (Russell and Norvig, 1995, pp.163-165,
592; Biermann, 1986, pp.134-135; Hawthorne, 2014; Beall and Restall, 2014;
Hume, 1777, section 4, part 2).

### Text Classification
Text classification is the NLP task of assigning a category to an input from a
predefined set of classes (Sebastiani, 2002, p.1; Manning et al, 2009, pp.256-258;
Manning and Schütze, 1999, pp.575-576).
Particular to our case, the documents will be natural language conversational user input,
and the set of categories will be the macro categories of issues that have been
extracted from [the concerns checklist (CC)](#ConcernsChecklist) version of the
questionnaire (see above).
This task is turned into a supervised machine learning task by training
a model over a set of document-category pairs (Sebastiani, 2011, slide 7, 13).

The internal representation of each document to the classifier is a sparse
vector representing the features or characteristics of the document relevant
to the classification task.
These features can be, for example, the occurrence or non-occurrence in the
document of certain words (set-of-words approach) (Sebastiani, 2011, slide 66;
Manning et al, 2009, pp.271-272)[^sentiment].

[^sentiment]: The task of sentiment analysis was also deemed
important to the assessment of the user distress level with respect to issues
the user would be discussing with the chatbot. But became less important as
requirements where clarified (Chapter 3). The reader is directed to Liu, 2010, p.628, 637--;
and Ding, Liu and Yu, 2008 for more information on the topic.

### Open Source NLP Libraries

The open source NLP libraries that were considered as part of the project were
the ones that could easily be used with the Python programming language (in line with
the author's personal aims). This excluded, for example, the
OpenNLP Apache library, due to its focus on Java (Apache, 2015).
Another requirement was to have a common NLP base for all PEACH subprojects.
It was decided, primarily based on the experience of the members of the
PEACH team that had previously worked with NLP to use the Natural Language Tool-Kit
(NLTK) as a baseline (Bird et al, 2014).
Its focus on Python also made it a better solution with respect to the author's aims than
other suites such as the Stanford CoreNLP, due to lack of extensive Python bindings
from the Java implementation (Manning et al, 2014; Smith, 2014).
The present project thus made primarily use of the NLTK, and packages built on
top of it.
The NLTK exposes a range of natural language corpora and ready available implementations of
various types of classifiers, with an intuitive API (Bird et al, 2014, Chapter 6 see also
Manning et al, 2009, pp.271--)[^nodata].

[^nodata]: An obvious problem
with the present project is that no data of the relevant shape was available whatsoever.
See Chapter 5 and Appendix C for more information on the solution adopted.

## Generating Chatbot Brain Data

Given the conclusion of using a software package that works based on input patterns matchers
and output templates, which have to be hard coded, investigation began into automated
generation of matchers and templates. In particular, the automated generation of
pattern matchers made of English words close in meaning was investigated.

### The word2vec Algorithm

One way to automatically generate synonyms is by looking at regularities
discovered in the use of English words through unsupervised learning. This
is at the core of what the word2vec algorithm does: it discovers regularities
based on the position words are used in sentences (Ellenberg, 2016). For each word in the training
data (the vocabulary) the algorithm constructs a vector representing the positional
regularities discovered between words in the training data.

Similarities between the use of words can be then expressed in geometric-algebraic
terms as the cosine distance between vectors representing the words
(Mikolov, 2015; McCormick, 2016b). This sort of similarity seemed like an interesting
way to automatically generate synonyms for use with the chatbot.

### The Gensim Library

The Gensim library (Rehurek, 2014; Rehurek and Sojka, 2010)[^gensimlibrary]
is another natural language processing tool for use with Python specialized
in document similarity computations and related tasks. A very straightforward way to
use word2vec for the stated purpose is through Gensim (McCormick, 2016a).

[^gensimlibrary]: <https://github.com/RaRe-Technologies/gensim>

Since the sort of data relevant to the training of a word2vec model for the purpose
of synonym generation did not require domain-specific data, but was in fact best
gathered through general English sources, the model that was used for the synonym
generation task was a model that had been pre-trained over a significant amount of
Google News data (McCormick, 2016a).

### Alternatives

WordNet (Fellbaum, 2005) is another alternative. There are ways to
interface with it via the NTLK (see http://www.nltk.org/howto/wordnet.html), but,
as mentioned in the chatbot brain discussion
above, some chatbot frameworks other than the one used for the current implementation
(RiveScript) already offer similar WordNet integrations out of the box (see for
ChatScript: Wilcox, 2016b, pp.10-11).
For other options into using dictionary definitions to extract
synonyms the reader is redirected to Wang and Hirst, 2012.

## Conclusion

This concludes the preliminary investigation section of the report. This chapter
covered similar existing technologies and background reading into the solution domain.
