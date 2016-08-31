made: YAML.md chapter1.md chapter2.md chapter3.md chapter4.md chapter5.md chapter6.md appendix.md
	pandoc --standalone --smart --normalize --number-section YAML.md chapter1.md chapter2.md chapter3.md chapter4.md chapter5.md chapter6.md appendix.md -o made.latex -f markdown -t latex --natbib
latex: made.latex
	pdflatex made.latex
bibl: made.latex made.aux
	bibtex made.aux  && \
	pdflatex made.latex && \
	pdflatex made.latex
