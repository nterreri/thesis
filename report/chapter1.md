\pagenumbering{arabic}
# Introduction
> "Pointing will still be the way to express nouns as we command our machines;
> speech is surely the right way to express the verbs."

**Frederick Brooks, 1995**

## The Problem
According to the 2014 National Cancer Patient Experience Survey National Report,
 only slightly over 20% of cancer patients across the UK reported having been
offered an assassment and care plan specific to their personal circumstances over the past couple of years
(Quality Health, 2014, p. 114).
In an effort to increase the number of cancer patients who received such
assessments, Macmillan Cancer Support piloted the Holistic Needs Assessment
(HNA) questionnaire and health plan in 2008 (Macmillan, [Holistic Needs Assessment][hna]).
This is a self-assessment questionnaire where the patient identifies
what their concerns are from a range of personal, physical, emotional and practical
issues they may be facing in their lives in relation to their condition.
The completion of the questionnaire is followed by the creation of a care plan
through a consultation with a clinician, with further advice and referrals as
needed. Macmillan began trialing an electronic version of the questionnaire in
2010, progressively extending provision of the eHNA to more and more sites
(Mac Voice, 2014).

This project is a about the use of an intelligent conversational system
to gather further information about the patient's concerns through an electronic
self-assessment tool, ahead of the creation of a patient care plan. This is primarily
an attempt at introducing the conversational User Interface in electronic
health applications generally, investigate related natural language processing and
tasks, and in particular explore the applicability of
computer advisors to Macmillan's eHNA in a growing effort to improve the
quality of support cancer patients receive across the UK.

Intelligent conversation systems have enjoyed an increasing amount of media
attention over the last year[^aiarticles]. With applications of artificial
intelligence to using natural language inputs for different purposes, including
general purpose mobile device interfaces[^viv]. Furthermore, several technology
companies have started offering "Artificial Intelligence as a Service" products.
Among these are BloomsburyAI (founded at UCL) and bespoken companies such as
Google and Microsoft[^aiaas]. This appears indicative of the fact that chatbot
and natural language processing technologies have reached a level of maturity
comparable to that achieved years ago by haptic technology, that we find almost ubiquitously
in human-computer interfaces and everyday use of computing devices today.

This project is part of PEACH: Platform for Enhanced Analytics and Computational
Healthcare (Project PEACH, 2016). PEACH is a data science project that
originated at University College London (UCL) in 2016 that sees Master level
candidates working together on the data platform and on related projects. With
more than twenty students across multiple Master courses, it is one of the
largest student projects undertaken in recent years at UCL, and it is part of
a long-term strategy to bring the UCL Computer Science department and the UCL Hospital
closer together.

The scope of the present report is limited to the architecture and implementation of the chatbot
system, as opposed to a complete user-facing product: the complete application is a joint effort of four
members of PEACH,
with distinct concerns being assigned to different members of the team. The
author of the present document is tasked with design and implementation of the core system backend. The
other members of the chatbot team include: Andre Allorerung (MSc SSE) as the
technical team lead who also oversees of the integration of the system with
the resources available to PEACH and the data storage system that will persist
information gathered through the chatbot system. Rim Ahsaini (MSc CS)
working on a specialized search engine for resources that may interest and help
support cancer patients based on their concerns (to be available both through
conversation with the chatbot and independently), Deborah Wacks (MSc CS) as lead
UX designer working on the implementation of a webserver through which deliver
 the chatbot and search engine to users.

[^aiarticles]: Numerous articles, among which: (The Economist, 2016),
(Berger, 2016), (Knowledge@Wharton, 2016), (Finextra Research, 2016)
[^viv]: (Viv, 2016), (Dillet, 2016)
[^aiaas]: (Pandorabots, 2016), (Riedel et al, 2016), (Microsoft Cognitive Services, 2016)

[hna]: http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/macmillansprogrammesandservices/recoverypackage/holisticneedsassessment.aspx
"Holistic Needs Assessment"

[^ehna]: <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/macmillansprogrammesandservices/recoverypackage/electronichollisticneedsassessment.aspx>, <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/newsandupdates/macvoice/winter2014/introductiontoehnaandcareplanning.aspx>, <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/newsandupdates/macvoice/winter2014/transformingcareusingehna.aspx>, <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/newsandupdates/macvoice/winter2014/developingtheehna.aspx>

## Project goals and personal aims
The main project goal is the delivery of a basic but easy to extend and modify
chatbot software system, specifically targeted at assisting with the identification
and gathering of information around cancer patient issues, modelled after the
Concerns Checklist (CC) electronic questionnaire form (NCSI, 2012).
Finally, one of the major challenges with eHealth problems is represented by
having to hand confidential patient data (as will be discussed in Chapter 2 of
this report). Summarily:

- Design and implement a chatbot architecture tailored to the issues surrounding
software systems in healthcare (in particular around treatment of sensitive patient data)
- To integrate with a specialized search engine (developed by another member of the team)
- To explore other applications of NLP that could be useful to extract information from natural language data.
- To implement a chatbot brain using open source technology.
- To develop the system with Macmillan eHNA as the main reference.

Personal goals of the author include:

- Learning Python in an effort to gain exposure to a new programming language
- Leverage the author's background in computational linguistics, and explore
the field of natural language processing
- Learn about applications of machine learning to natural language processing
- Improve software engineering skills by applying best agile methodology practices

## The project approach methodology
An agile methodology approach was adopted for the project, in line
with the author's stated interests. This meant maximizing time spent outside of
meetings, save for where communication between team members and others was required.
The project was paced in weekly iterations where aspects of the system to implement
would be selected from a backlog to be delivered for the next week, in consultation
with Dr Ramachandran who acted as the client for ever project connected with PEACH (Beck and Andres, 2014, pp.46-47).
Great emphasis was also put on testing as part of deveopment, in particular the discipline
of Test Driven Development.

A top-down system design and implementation was also adopted, with the next largest system
abstraction being priorized first in order to always have a
working system being progressively refined. These methodology guidelines where established in accordance with
the reccomendations of Brooks (1995, pp.143-144, 200-201, 267-271), Martin (2009, pp.121-133; 2003, chapter 2, 4, 5)
and Beck (Beck and Andres, 2014; Beck et al, 2001).

## Report overview
This report is structured as follows:

- Chapter 2 provides more extensive background into the
NLP and chatbot open source resources that were explored.
- Chapter 3 describes the requirements as gathered through the contacts in healthcare
and the Macmillan charity available to PEACH.
- Chapter 4 details the system architecture, design and the implementation,
highlighting its current limitations and design.
- Chapter 5 discusses the benefits of TDD to systems design, how system testing
was done as part of development, and the evaluation of the machine learning component
of the system.
- Chapter 6 concludes with an evaluation of the project results, a review of
the effectivenss of the core tools used,
and reccomendations for the direction of future work on the system.