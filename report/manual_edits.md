# Edit 1:
~~~
- \usepackage[unicode=true, breaklinks]{hyperref}
- \usepackage{url}
- \usepackage{breakurl}

either:
- \usepackage{natbib}
- \bibliographystyle{abbrvnat}
or:
-\usepackage[backend=bibtex]{biblatex-chicago}
-\bibliography{bibl.bib}
~~~
The above has to be put the latex manually to get the urls to format properly
in the references.

# Edit 2:
~~~
\pagenumbering{roman}
{
\setcounter{tocdepth}{2}
\tableofcontents
}
~~~
The \pagenumbering{roman} command must appear around the above area.

# Edit 3:
~~~
%----------------------------------------------------------------------------------------
%	THESIS INFORMATION
%----------------------------------------------------------------------------------------


\def\authorname{}
\def\ttitle{}
\newcommand*{\supervisor}[1]{\def\supname{#1}}
\newcommand*{\thesistitle}[1]{\def\@title{#1}\def\ttitle{#1}}
\newcommand*{\examiner}[1]{\def\examname{#1}}
\newcommand*{\degree}[1]{\def\degreename{#1}}
\renewcommand*{\author}[1]{\def\authorname{#1}}
\newcommand*{\addresses}[1]{\def\addressname{#1}}
\newcommand*{\university}[1]{\def\univname{#1}}
\newcommand*{\department}[1]{\def\deptname{#1}}
\newcommand*{\group}[1]{\def\groupname{#1}}
\newcommand*{\faculty}[1]{\def\facname{#1}}
\newcommand*{\subject}[1]{\def\subjectname{#1}}
\newcommand*{\keywords}[1]{\def\keywordnames{#1}}
\newcommand{\HRule}{\rule{\linewidth}{0.5mm}} % New command to make the lines in the title page

\thesistitle{A Conversational Chatbot Architecture for eHealth Systems} % Your thesis title, this is used in the title and abstract, print it elsewhere with \ttitle
\supervisor{Harry \textsc{Strange}} % Your supervisor's name, this is used in the title page, print it elsewhere with \supname
\examiner{} % Your examiner's name, this is not currently used anywhere in the template, print it elsewhere with \examname
\degree{MSc Computer Science} % Your degree name, this is used in the title page and abstract, print it elsewhere with \degreename
\author{Niccoló \textsc{Terreri}} % Your name, this is used in the title page and abstract, print it elsewhere with \authorname
\addresses{} % Your address, this is not currently used anywhere in the template, print it elsewhere with \addressname

\subject{Computer Science} % Your subject area, this is not currently used anywhere in the template, print it elsewhere with \subjectname
\keywords{} % Keywords for your thesis, this is not currently used anywhere in the template, print it elsewhere with \keywordnames
\university{\href{http://www.university.com}{University College London}} % Your university's name and URL, this is used in the title page and abstract, print it elsewhere with \univname
\department{\href{http://department.university.com}{Department of Computer Science}} % Your department's name and URL, this is used in the title page and abstract, print it elsewhere with \deptname
\group{\href{http://researchgroup.university.com}{Research Group Name}} % Your research group's name and URL, this is used in the title page, print it elsewhere with \groupname
\faculty{\href{http://faculty.university.com}{Faculty of Engineering}} % Your faculty's name and URL, this is used in the title page and abstract, print it elsewhere with \facname

\hypersetup{pdftitle=\ttitle} % Set the PDF's title to your title
\hypersetup{pdfauthor=\authorname} % Set the PDF's author to your name
\hypersetup{pdfkeywords=\keywordnames} % Set the PDF's keywords to your keywords

\begin{document}

\pagestyle{plain} % Default to the plain heading style until the thesis style is called for the body content

%----------------------------------------------------------------------------------------
%	TITLE PAGE
%----------------------------------------------------------------------------------------

\begin{titlepage}
\begin{center}

\includegraphics{UCL-logo-new.png} % University/department logo - uncomment to place it

{\scshape\LARGE \univname\par}\vspace{0.1cm} % University name
\textsc{\Large MSc Computer Science}\\[0.5cm] % Thesis type
\facname\\\deptname\\[0.5cm] % Research group name and department name

\HRule \\[0.4cm] % Horizontal line
{\Large \bfseries \ttitle\par}\vspace{0.4cm} % Thesis title
\HRule \\[1.0cm] % Horizontal line

\begin{minipage}[t]{0.4\textwidth}
\begin{flushleft} \large
\emph{Author:}\\
\href{http://www.johnsmith.com}{\authorname} % Author name - remove the \href bracket to remove the link
\end{flushleft}
\end{minipage}
\begin{minipage}[t]{0.4\textwidth}
\begin{flushright} \large
\emph{Supervisor:} \\
\href{http://www.jamessmith.com}{\supname} % Supervisor name - remove the \href bracket to remove the link
\end{flushright}
\end{minipage}\\[2cm]


{\today}\\[4cm] % Date

\vfill
\end{center}
\end{titlepage}
~~~
Frontpage edit, appears in place of begin{document} and maketitle
