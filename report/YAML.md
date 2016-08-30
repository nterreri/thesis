---
documentclass: report
fontsize: 12pt
linestretch: 1.5
papersize: a4
classoption: twoside

title: A Conversational Chatbot Architecture for eHealth Systems
author: Niccoló Terreri, Master of Science in Computer Science candidate

toc: true
secnumdepth: 2

bibliography: bibl.bib
header-includes:
    - \usepackage{pbox}
    - \usepackage[table,xcdraw]{xcolor}
    - \usepackage{url}
    - \usepackage{breakurl}
    - > %  \subsection{The Title Page}
%
%  \begin{macro}{\maketitle}
%    Making a title page is non-trivial, especially for a display title page; things are
%    carefully synchronised here so don't change randomly.
%
%    (said Russel, before Ian changed something randomly)
%    We want the author variable for the "I have not plagiarised everything" declaration.
%    We also need a new department variable.
%    \begin{macrocode}
\def\department#1{\gdef\@department{#1}}
\def\@department{\@latex@warning@no@line{No \noexpand\department given}}
\renewcommand \maketitle{%
    \setcounter{page}{1}%
    \thispagestyle{empty}%
    \@maketitle%
    \setcounter{footnote}{0}%
    \let \thanks = \relax%
    \gdef \@address{}%
    \gdef \@thanks{}%
    %\gdef \@author{}%
    \gdef \@department{}%
    \gdef \@title{}%
    \let \maketitle = \relax%
}
%    \end{macrocode}
%  \end{macro}
%  \begin{macro}{\@maketitle}
%  Create a separate title page with the usual material on it.
%    \begin{macrocode}
\newcommand \@maketitle{%
    \newpage%
    \null%
    \vspace*{5em}%
%%%
%%%  Julia Schnabel wanted the following added.
%%%
%%%    \renewcommand{\baselinestretch}{2.5}
%%%    \small
%%%    \normalsize
%%%
    \begin{center}%
        {\huge \bfseries \@title}\\[5em]%
        {\Large \itshape \@author}\\%
    \end{center}%
    \vfill%
    \begin{center}%
    A dissertation submitted in partial fulfillment \\
    of the requirements for the degree of \\
    \textbf{\@degree@string} \\
    of \\
    \textbf{University College London}.
    \end{center}%
    \vspace{2em}%
    \begin{center}%
    \@department \\
    University College London\\
    \end{center}%
    \vspace{2em}%
    \begin{center}%
    \@date%
    \end{center}%
    \if@twoside %
      \newpage%
      ~\\
      \newpage%
    \fi
}
%    \end{macrocode}
%  \end{macro}
%

abstract: > This project is about the architecture and design of the core backend
of a conversational agent for eHealth applications. It is part of a larger team
effort aiming at the delivery of a complete user-facing application, specifically
 modelled after Macmillan's eHNA system for cancer patients in the UK.
It involved research in open source chatbot technologies, and related NLP tasks
such as text categorization, as well as the design, testing and basic implementation
of the system. The developement was carried out in weekly iterations in continuous
consultation with a client for the University College London Hospital.
The project delivered an extensible implementation in the form of a software
package meant for use with an external system of delivery to the user (a
  webserver in the final product of the team effort).

---
