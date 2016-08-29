# Requirements Gathering {#Chapter3}

This chapter describes the full problem statement, and the way the list of
requirements was produced and agreed on by the stakeholders. Recall that the scope
of this project is the architecture of the core system, not a full end-to-end
implementation, which is the objective of the PEACH chatbot team. UI/UX design
does not fall within the scope, and neither does a data persistence strategy
(datastore solutions etc) nor a full deployment plan.

## Building the Right System

The problem to be solved is an introduction of the conversational UI into the
eHealth sphere taking into account UK legislation over the use of patient data.
This is inspired by the Macmillan Cancer Support chartiy eHNA system, that is
just finishing its extended trial period and aims at extending the coverage
of support cancer patients receive in the UK with personalized care plans
(Mac Voice, 2014). The introduction of this UI is intended to help streamlining
the process of filling out the eHNA for the patients, and gather insight into the
patient issues ahead of the in-person review, shortening the time
needed for the creation of the care plan in oder to facilitate its creation,
extending the coverage to more cancer patients, in an effort to improve the quality
of care they receive across the UK.

In this initial phase of the project, the key objective is the engineering
of an extensible architecture, which is expected to be subject of significant
modifications as the software is tweaked in following iterations to accommodate
for changes in the nature of the problem, and in the technological landscape. In
other words, focus on minimization of technical debt (Fowler, 2003).

As discussed in the previous chapter, there are increasingly interesting developments
happening in the use of recurrent neural networks for chatbot applications, but
at the time of writing the technology is still immature, while an older model
primarily based around the gathering of specific set of parameters from the user
through conversation, achieved through hard-coded input patterns matchers and
response templates, that has matured over the last fifteen years, is already being
used in professionally developed software (Vinyals and Le, 2015; Jurafsky and Martin, 2009, Chapter 24;
Wilcox, 2011).

The aim of the project is therefore to engineer this architecture and provide
a basic proof-of-concept implementation of the system, with *the core of the
chatbot system being an easy to replace implementation detail* rather than the core focus of
the project effort.

Note that, in accordance with agile methodology principles (as per the author's aims), "architecture"
here does not refer *exclusively* to the production of documents such as class and
component diagrams, or other UML artifacts (Object Management Group, 2015); but rather
the *primary* document ideintified as the technical design document produced by the engineering activity is the source code.
Just like in other engineering industries, software engineers design a meticulously
crafted document that is then passed on to the manufacturing staff, requiring no further input from
the designers to produce a concrete artifact. The manufacturing "staff", in this
case, being compilers, linkers, interpreters and virtual machines (Reeves, 2001).

## Requirements Gathering

The project was first proposed by Dr Ramachandran, who also organized a meeting
between Macmillan staff (including the technical lead of the Macmillan eHNA,
Andrew Brittle). The original plan was to have both the author and Rim Ahsaini
working on the core chatbot system, with the full questionnaire being completed
by the user through interaction with the chatbot.

During the first meeting, the idea of a specialized search engine
emerged, with Rim Ahsaini interested in taking charge of that system. From there
came also the idea of integration between the two systems, with the chatbot
system having to decide whether to make a query during the interaction with the
user.

During the second meeting, when the team was given a full demo of the eHNA
system. It emerged that users tend to only select a relatively small amount
of concerns, between two and six depending on what type of cancer they have.
Furthermore, the questionnaire itself takes a relatively small amount of time.
The requirements for the core chatbot system were afterwards revised to
allow the user to first fill out the questionnaire before talking to the chatbot
in order to keep the efficiency of the eHNA software model.

The rest of the requirements were mostly decided by the author and project
supervisor on the basis of what would be most useful to investigate from a technical
point of view looking at the future of the project. Objectives were decided on
a weekly basis, in line with agile methodology.

## Requirements Listing

\renewcommand{\arraystretch}{2}
\setstretch{1.0}
\begin{table}[]
\centering
\caption{Requirements table}
\label{my-label}
\begin{tabular}{|l|l|l|} \hline
\textbf{Label} & \textbf{Requirement}                                       & \textbf{Priority}\\
\hline
RQ0            & \pbox{25cm}{The chatbot will categorize userinput into\\
                    predefined concern categories}                          & {\color[HTML]{FE0000} Must}   \\
\hline
RQ1            & \pbox{25cm}{The chatbot will generate replies aimed at\\
                    asking the user for more details, and gather \\
                    information on concern raised}                          & {\color[HTML]{FE0000} Must}   \\
\hline
RQ2            & \pbox{25cm}{The chatbot will decide whether to reply with \\
                    a generative or non-generative reply}                    & {\color[HTML]{FE0000} Must}\\
\hline
RQ3            & \pbox{25cm}{The chatbot will decide whether the generative \\
                reply should be provided as the result of a query or bot-brain\\
                generated reply}                                            & {\color[HTML]{F8A102} Should}\\
\hline
RQ4            & \pbox{25cm}{The chatbot will model the data gathered appropriately\\
                            for durable storage in datastore}                  & {\color[HTML]{FE0000} Must}   \\
\hline
RQ5            & \pbox{25cm}{The chatbot will interface with the search/query \\
                engine component of the larger system it is part of}         & {\color[HTML]{F8A102} Should} \\
\hline
RQ6            & \pbox{25cm}{The chatbot may performsentiment analysison user \\
                input}                                                       & {\color[HTML]{32CB00} Could}  \\
\hline
RQ7            & \pbox{25cm}{The chatbot may use the preprocessed input \\
                (categorization/sentiment analysis) to inform the bot-brain} & {\color[HTML]{F8A102} Should} \\
\hline
RQ8            & \pbox{25cm}{The chatbot may interface with server side request\\
                handling logic}                                              & {\color[HTML]{F8A102} Should} \\
\hline
RQ9            & \pbox{25cm}{The chatbot mayreceive/return queries in the form of\\
                JSON/XML documents}                                          & {\color[HTML]{32CB00} Could}  \\
\hline
RQ10           & \pbox{25cm}{The chatbot may be deployed outside the context of \\
                        eHNAs as an independently accessible service}        & {\color[HTML]{32CB00} Could}  \\
\hline
RQN0           & \pbox{25cm}{The chatbot will not have an offline/cached \\
                operating mode}                                              & Will not  \\              
\hline
\end{tabular}
\end{table}}
\setstretch{1.5}

## Use Case Diagram
