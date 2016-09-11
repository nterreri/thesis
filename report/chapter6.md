# Conclusion

>> *"When are you done? Since design is open-ended, the most common answer to that
>> question is 'When you're out of time.'"*[^connell]

[^connell]: McConnell, 2004, p.76.

This chapter reviews the project goals and aims, finally, the author expresses
his thoughts on the project going forward.

## Project Goals Review

The goals and aims introduced in Chapter 1 are here reviewed.
All references to "RQ" below refer to the requirements
table in Chapter 3. The project achieved all key requirements and
several additional requirements[^missedreqs].

1. Design and implement a chatbot architecture tailored to the issues surrounding
software systems in healthcare.

As discussed in Chapter 2, there are crucial concerns around the sensitivity
of patient data that have driven the implementation decision not to employ third
party APIs for the processing of natural language user input, even where this
may have significantly simplified the chatbot brain implementation task (see RQNF0, RQ1).

2. To integrate with a specialized search engine.

While development over the current project had terminated before the search engine
was completed, a clear reference on how to integrate between the systems had
been provided (and materialized in the integration testing *MockSearch* class
and shown integrated at the postprocessing stage; see also RQ5 and Section 5.2).

3. To explore other applications of NLP that could be useful to extract information from natural language data

NLP was used to investigate a hybrid approach to chatbot
technology, making use of both rule or grammar based technologies and machine
learning with the categorizer (RQ0, RQ2, RQ3). Secondly, NLP was used for the synonym generation (RQ8).

4. To implement a chatbot brain using open source technology.

As mentioned, a chatbot brain was implemented in RiveScript, after reviewing
the available choice of technologies (RQNF0, RQ1).

5. To develop the system with Macmillan eHNA as the main reference.

As discussed in Chapter 2, it was with reference to the CC used in one of Macmillan
eHNA forms that was used for the concrete implementation of the topics in the
"concerns" sub-package (RQNF1).

[^missedreqs]: The requirements not covered by this list were not selected as strictly necessary
for the project success (*May* under MoSCoW: RQ8, RQ9).
RQ7 and RQ8 echo the initial uncertainty with respect to the
technologies the other team members were going to use, later the whole team used
Python making such concerns redundant, however they are left in the list as they
would need to be considered should the technologies around the chatbot component
change in the future.

## Personal Aims Review

1. Learning Python in an effort to gain exposure to a new programming language.

The Python programming language was used to implement the majority of the project
with the exception of the specialized RiveScript language used to define the
pattern matchers in the chatbot brain.

2. Leverage the author's background in computational linguistics, and explore
natural language processing.

As mentioned in the previous section, the author investigated the use of NLP
to text classification and synonym generation (in addition to significantly
more reading in NLP than resulted useful for the project).

3. Learn about applications of machine learning in natural language processing.

The categorization component involved the development of supervised learning
language models, while the synonym generation made use of a pre-trained unsupervised
learning model.

4. Improve software engineering skills by applying best agile methodology practices.

A significant amount of reading into agile methodology practices was demonstrated in the
project in the system design, implementation and testing.

## Future Work

The implementation of
a conversation agent provided is incomplete and inarticulate and neither
of the two machine learning components of the system score excellently well.
However, the primary goal of the project was to lay the ground work,
research and software design for further iterations of the project;
because of this, more time was spent on the implementation of the supporting
infrastructure than on the chatbot brain and machine learning components
implementation.
The design of the system
is modular, loosely coupled and easily extensible. The coverage of unit and
integration tests provides confidence to any programmer continuing the work here
started that the system still displays all behaviours it did when it was delivered
at the end of this project, no matter what changes were made during revision and
expansion.

The "plugin" model makes the system independent of the IO device that delivers it
which may be in the future a mobile app,
a CLI client, public chat platforms or anything else. It is the caller's
responsibility to import this project's packages and modules, and to use them
as documented in the unit and integration tests. Similarly, the data models gathering conversation and
user concerns data should be used in a similar fashion: independently of the
durable storage solution adopted (whether this is SQL, NoSQL, host filesystem
or any other)[^facade].

[^facade]:  Although both consumers of the
system would benefit from the creation of a unified façade instead of having dependencies
across multiple subpackages of the system (Gamma et al, 1995, Chapter 4.5).  

There are various ways in which the project could be extended. First of all,
the author advises exploring an open source framework different from RiveScript
given the limitations discovered within it during this project (or if the client believes
it appropriate using third party remote APIs). See Appendix A (A.2.2)
for instructions on how to replace RiveScript, and
Chapter 2 for a discussion of alternatives.

Secondly, the categorizer module (to which only a week was dedicated during this
project) could be made more robust to use sequence classifiers, potentially
extending the NLTK open source project (see Appendix D for
initial research in this direction). Eventually, this would become part of the
preprocessing layer to inform the input to the rule-based chatbot engine with the
output of a machine learning component (as outlined in Section 2.4).
Third, generation of RiveScript (or similar specialized languages) could be investigated
following up on the initial results with the synonym generation package (Section 2.5).

Finally and most importantly, it may be desirable to investigate information-retrieval techniques to
extract information from the conversation data gathered during conversations,
not necessarily as a way to fill the questionnaire (as originally intended, see Section 3.2)
but to focus on producing summaries of user concerns for the clinician.
One of the biggest obstacles to providing cancer patients with better care is the
time it takes to create a care plan. Through the extraction
of information about the patient's concerns ahead of time, it may be possible
to shorten the duration of the care plan appointments, and therefore allow
a larger number of patients to be provided with a care plan. Of course, it would
still be necessary to first gather relevant data through the conversational UI
or a different solution.
