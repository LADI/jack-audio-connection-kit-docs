.PHONY: html html-mkdir

html: ./html/index.html
html: ./html/jackdbus-packaging.html
html: ./html/suggested-packaging-approach.html

html-mkdir:
	@mkdir -vp html

./html/index.html : ../jackdbus/README.adoc html-mkdir
	asciidoc -b html5 -a data-uri -a icons --theme ladi -o $@ $<

./html/%.html : ./%.adoc html-mkdir
	asciidoc -b html5 -a data-uri -a icons --theme ladi -o $@ $<
