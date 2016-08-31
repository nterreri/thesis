# Conclusion

This chapter reviews the project goals and aims, finally, the author expresses
his thoughts on the project going forward, informed by a critical review of the
open source tools used. All references to "RQ" below refer to the requirements
table in Chapter 3.

## Project Goals Review

- **Design and implement a chatbot architecture tailored to the issues surrounding
software systems in healthcare (in particular around treatment of sensitive patient data)**

As discussed in Chapter 2, there are crucial compliance concerns around the sensitivity
of patient data that have driven the implementation decision not to employ third
party APIs for the processing of natural language user input, even where this
may have significantly simplified the chatbot brain implementation task (see RQNF0, RQ1).
From this it is possible to see the project achieved all key requirements and
several additional requirements[^missedreqs].

- **To integrate with a specialized search engine (developed by another member of the team)**

While development over the current project had terminated before the search engine
was completed, a clear reference on how to integrate between the systems had
been provided (and materialized in the integration testing "MockSearch" class
and shown integrated at the postprocessing stage; see also RQ5).

- **To explore other applications of NLP that could be useful to extract information from natural language data.**

The current project has used NLP to investigate a hybrid approach to chatbot
technology, making use of both rule or grammar based technologies and machine
learning with the categorizer (RQ0, RQ2, RQ3). Secondly, NLP was used for the synonym generation (RQ8).

- **To implement a chatbot brain using open source technology.**
As mentioned, a chatbot brain was implemented in RiveScript, after reviewing
the available choice of technologies given the restrictions on patient data (RQNF0, RQ1).

- **To develop the system with Macmillan eHNA as the main reference.**

As discussed in Chapter 2, it was with reference to the CC used in one of Macmillan
eHNA forms that was used for the concrete implementation of the topics in the
"concerns" subpackage (RQNF1).

[^missedreqs]: The requirements not covered by this list were not selected as strictly necessary
for the project success (*May* under MoSCoW: RQ8, RQ9).
RQ7 and RQ8 echo the initial uncertainty with respect to the
technologies the other team members were going to use, later the whole team used
Python making such concerns redundant, however they are left in the list as they
would need to be considered should the technologies around the chatbot component
change in the future.

## Personal Aims Review

- **Learning Python in an effort to gain exposure to a new programming language**

The Python programming language was used to implement the majority of the project
with the exception of the specialized RiveScript language used to define the
pattern matchers in the chatbot brain.

- **Leverage the author's background in computational linguistics, and explore
the field of natural language processing**

As mentioned in the previous section, the author investigated the use of NLP
to text classification and synonym generation, in addition to significantly
more reading in NLP than resulted useful for the project.

- **Learn about applications of machine learning to natural language processing**

The categorization component involved the development of supervised learning
language models, while the synonym generation made use of a pre-trained unsupervised
learning model.

- **Improve software engineering skills by applying best agile methodology practices**

A significant amount of reading into agile methodology practices was demonstrated in the
project in the system design, implementation and testing.

## Future Work

The primary goal of the project was to lay the ground work, architecture and
research for further iterations of the project. As emphasized throughout Chapter 4,
the system was designed for extension and modification. The coverage of unit and
integration tests provides confidence to any programmer continuing the work here
started that the system still displays all behaviours it did when it was delivered
at the end of this project, no matter what changes were made during revision and
expansion.

The "plugin" model makes the system independent of the IO device that delivers it
(in the case of the group project, a webserver) which may be in the future a mobile app,
a CLI client, or a Java Swing interface. It is, in fact, the caller's
responsibility to import this project's packages and modules, and to use them
as documented in the test cases. The data models gathering conversation and
user concerns data should be used in a similar fashion: independently of the
durable storage solution adopted (whether this is SQL, NoSQL, host filesystem
or any other) it should be the caller's responsibility to import this project
to serialize the data.

### Evaluation of Technologies Used
#### RiveScript
While the choice of RiveScript as the chatbot brain framework was sensible at the
time given the prior research, the author reccomends other options are investigated
during the next iteration, for the following reasons.

The undocumented problems encountered with it during
development, its limitations with respect to matching semantic patterns of input
(unlike others like ChatScript), the chatbot brain implementation provided with the
current project is not very extensive.

#### NLTK and Gensim
The NTLK proved to be a versatile tool, used primarily in the categorization
and message processing modules. Its main limitation is the lack of sequence
classifier support (as lamented in Chapter 2). For this reason it is recommended
that alternatives that offer such models are investigated during the next
iteration (see for example Schreiber et al, 2016).

The use made of Gensim in the current iteration was very minimal, but this tool
seems so highly specialized that it is difficult to find alternatives to it.
