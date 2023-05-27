.PHONY: html html-mkdir

html: ./html/index.html
html: ./html/packaging.html
html: ./html/installation-filesystem-layout.html

html-mkdir:
	@mkdir -vp html

./html/index.html : ../jackdbus/README.adoc html-mkdir
	asciidoc -b html5 -a data-uri -a icons --theme ladi -o $@ $<

./html/%.html : ./%.adoc html-mkdir
	asciidoc -b html5 -a data-uri -a icons --theme ladi -o $@ $<
