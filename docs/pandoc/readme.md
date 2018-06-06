sudo apt-get install pandoc
sudo apt-get install miktex
sudo apt-get texlive-xetex texlive-luatex pandoc-citeproc wkhtmltopdf

pandoc infile.md -o outfile.pdf
pandoc infile.md -o outfile.pdf --latex-engine=xelatex

pandoc infile.md -o outfile.pdf --latex-engine=xelatex -V mainfont="SimSun"

https://github.com/tzengyuxio/pages/tree/gh-pages/pandoc