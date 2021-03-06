---
documentclass: report
fontsize: 12pt
linestretch: 1.5
papersize: a4
classoption: twoside

title: A Conversational Chatbot Architecture for eHealth Systems
author: Niccoló Terreri

toc: true
secnumdepth: 2

header-includes:
    - \usepackage[a4paper]{geometry}
    - \usepackage{pbox}
    - \usepackage[table,xcdraw]{xcolor}
    - \usepackage[backend=bibtex]{biblatex-chicago}
    - \bibliography{MSC16.bib}

abstract: "This project is about the architecture and design of the core backend
of a conversational agent for eHealth applications. It is part of a larger team
effort aiming at the delivery of a complete user-facing application, specifically
 modelled after Macmillan's eHNA system for cancer patients in the UK.
It involved research in open source chatbot technologies, and related NLP tasks
such as text categorization, as well as the design, testing and basic implementation
of the system. The development was carried out in weekly iterations in continuous
consultation with a client for the University College London Hospital.
The project delivered an extensible implementation in the form of a software
package meant for use with an external system of delivery to the user (a
  webserver in the final product of the team effort)."

---
