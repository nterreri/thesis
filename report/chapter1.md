
# Introduction
>> *"Pointing will still be the way to express nouns as we command our machines;
>> speech is surely the right way to express the verbs."*[^brooks]

[^brooks]: Frederick Brooks, 1995, p.264.

\pagenumbering{arabic}

## The Problem
According to the 2014 National Cancer Patient Experience Survey National Report,
 only slightly over 20% of cancer patients across the UK reported having been
offered an assessment and care plan specific to their personal circumstances over the past couple of years
(Quality Health, 2014, p. 114).
In an effort to increase the number of cancer patients who receive such
assessments, Macmillan Cancer Support piloted the Holistic Needs Assessment
(HNA) questionnaire and health plan in 2008 (Rowe, 2014)[^hna].
This is a self-assessment questionnaire where the patient identifies
what their concerns are from a range of personal, physical, emotional and practical
issues they may be facing in their lives in relation to their condition.
The completion of the questionnaire is followed by the creation of a care plan
through a consultation with a clinician, with further advice and referrals as
needed. Macmillan began trialling an electronic version of the questionnaire in
2010, progressively extending provision of the eHNA to more and more sites
(Rowe, 2014).

This project is a about the use of an intelligent conversational system
to gather further information about the patient's concerns through an electronic
self-assessment tool, ahead of the creation of a patient care plan. This is primarily
an attempt at introducing the conversational User Interface in electronic
health applications generally, investigate related natural language processing (NLP) and
tasks, and in particular explore the applicability of
computer advisors to Macmillan's eHNA in an effort to improve the
quality of support cancer patients receive across the UK.

Intelligent conversation systems have enjoyed an increasing amount of media
attention over the last year (numerous articles among which: The Economist, 2016;
Berger, 2016; Knowledge@Wharton, 2016; Finextra Research, 2016; Yuan, 2016; French, 2016).
With applications of artificial
intelligence to using natural language inputs for different purposes, including
general purpose mobile device interfaces (Viv Labs, 2016; Dillet, 2016). Furthermore, several technology
companies have started offering "Artificial Intelligence as a Service" (AIaaS) products.
Among these are BloomsburyAI and other companies such as
Google and Microsoft (Pandorabots, 2016; Riedel et al, 2016; Microsoft Cognitive Services, 2016).
This appears indicative of the fact that chatbot
and natural language processing technologies are reaching a level of maturity
comparable to that achieved years ago by haptic technology, that we find almost ubiquitously
in human-computer interfaces and everyday use of computing devices today.

This project is part of PEACH: Platform for Enhanced Analytics and Computational
Healthcare (PEACH, 2016). PEACH is a data science project that
originated at University College London (UCL) in 2016 that sees Master level
candidates working together on the data platform and on related projects. With
more than twenty students across multiple Master courses, it is one of the
largest student projects undertaken in recent years at UCL, and it is part of
a long-term strategy to bring the UCL Computer Science department and the UCL Hospital
closer together.

The scope of the present report is limited to the architecture and implementation of the chatbot
system, as opposed to a complete user-facing product: the complete application is a joint effort of four
members of PEACH. The
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

[^hna]: <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/macmillansprogrammesandservices/recoverypackage/holisticneedsassessment.aspx>

[^ehna]: <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/macmillansprogrammesandservices/recoverypackage/electronichollisticneedsassessment.aspx>, <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/newsandupdates/macvoice/winter2014/introductiontoehnaandcareplanning.aspx>, <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/newsandupdates/macvoice/winter2014/transformingcareusingehna.aspx>, <http://www.macmillan.org.uk/aboutus/healthandsocialcareprofessionals/newsandupdates/macvoice/winter2014/developingtheehna.aspx>

## Project goals and personal aims
The main project goal is the delivery of a basic but easy to extend and modify
chatbot software system, specifically targeted at assisting with the identification
and gathering of information around cancer patient issues, modelled after the
Concerns Checklist (CC) electronic questionnaire form (NCSI, 2012; see Section 2.1.1).
Finally, one of the major challenges with eHealth problems is represented by
having to handle confidential patient data (Chapter 2.2). Summarily:

- Design and implement a chatbot architecture tailored to the issues surrounding
software systems in healthcare.
- To integrate with a specialized search engine (developed by another member of the team).
- To explore other applications of NLP that could be useful to extract information from natural language data.
- To implement a chatbot brain using open source technology.
- To develop the system with Macmillan eHNA as the main reference.

Personal goals of the author include:

- Learning Python in an effort to gain exposure to a new programming language.
- Leverage the author's background in computational linguistics, and explore
the field of natural language processing.
- Learn about applications of machine learning to natural language processing.
- Improve software engineering skills by applying best agile methodology practices.

## The project approach methodology
An agile methodology approach was adopted for the project, in line
with the author's stated interests. This meant minimizing communication overhead
and focusing on code, testing and value to the user.
The project was paced in weekly iterations where aspects of the system to implement
would be selected from a backlog to be delivered for the next week, in consultation
with Dr Ramachandran who acted as the client for every project connected with PEACH.
Great emphasis was also put on testing as part of development, in particular the discipline
of Test Driven Development.
A top-down system design and implementation was also adopted, with the next largest system
abstraction being prioritized first in order to always have a
working system being progressively refined (Brooks, 1995, pp.143-144, 200-201, 267-271; Martin, 2009, pp.121-133; 2003, chapter
 2, 4, 5; Beck and Andres, 2004, pp.46-47; Beck et al, 2001.)

## Report overview
This report is structured as follows:

- Chapter 2 provides more extensive background into the
chatbot and NLP open source resources that were explored.
- Chapter 3 describes the requirements as gathered through the contacts in healthcare
and the Macmillan charity available to PEACH.
- Chapter 4 details the system architecture, design and the implementation.
- Chapter 5 discusses the the system testing strategy, and the evaluation strategy of the machine learning components.
- Chapter 6 concludes with an evaluation of the project results
and recommendations for the direction of future work on the system.

\nocite{*}
