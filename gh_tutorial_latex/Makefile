MAIN=gh_tutorial.tex
DEPENDS=Makefile 

DIFFDIR=

#OS dependent
UNAME=$(shell uname)
ifeq ($(UNAME),Darwin)
REALPATH=grealpath
else
REALPATH=realpath
endif

TARGET=$(subst .tex,.pdf,$(MAIN))
TARGET_PRE=$(subst .tex,.dvi ,$(MAIN))
AUX=$(subst .tex,.aux,$(MAIN))
BASE=$(basename $(MAIN))

BIB=$(subst .tex,.bbl,$(MAIN))
LOG=$(subst .tex,.log,$(MAIN))
FLS=$(subst .tex,.fls,$(MAIN))

#PDFLTEX=platex
PDFLTEX=platex
BIBTEX=bibtex
DVIPDF=dvipdfmx
#map= -f /usr/share/texmf/fonts/map/dvipdfmx/hiraginomaru.map
#map= -f jfontmaps/hiragino/otf-hiragino.map
ifeq ($(UNAME),Darwin)
map= -f /opt/local/share/texmf-texlive/fonts/map/dvipdfmx/jfontmaps/hiragino/otf-hiragino.map 
else
map= -f /usr/share/texmf-dist/fonts/map/dvipdfmx/jfontmaps/hiragino/otf-hiragino.map
endif
#DVIPDF=dvipdfm

FILES=.files
DEPS=.deps
#BATCHMODE=-interaction batchmode
BATCHMODE=

PDFLTEX_FLAGS=$(BATCHMODE) --recorder -shell-escape
#PDFLTEX_FLAGS=$(BATCHMODE) -recorder

$(TARGET_PRE): $(MAIN) $(FORCE)
	$(MAKE) compile

pdf: $(TARGET)

$(TARGET): $(TARGET_PRE)
	$(DVIPDF) $(map) $(TARGET_PRE)


#draft:
#	$(MAKE) compile DRAFT=$(DRAF_COMMAND) 
#
compile:
	$(PDFLTEX) $(PDFLTEX_FLAGS) $(MAIN)

bib: $(BIB)

$(BIB): $(FORCE)
	$(BIBTEX) $(BASE)


$(FLS):
	$(MAKE) compile

$(FILES): $(FLS)
	grep INPUT $(BASE).fls | cut -b 7- | xargs -n1 $(REALPATH) --strip | sort -u | grep `$(REALPATH) $${PWD}` | sed "s#`$(REALPATH) $${PWD}`/##" | grep -v ".out" | grep -v ".aux" > $(FILES)
	@echo $(DEPENDS) | xargs -n1 >> $(FILES)
	mv $(FILES) .tmp
	sort -u .tmp > $(FILES)
	rm -f .tmp


$(DEPS): $(FILES)
	echo "$(TARGET_PRE): \\" > $@
	cat $(FILES) | sed "s/^/	/g" | sed "s/$$/\\\\/g" >> $@
	echo "	" >> $@

ifneq ($(MAKECMDGOALS),compile)
-include $(DEPS)
endif

ARC_CMMAND=zip -r
ARCHIVE=$(subst .tex,.zip,$(MAIN))
#ARC_CMMAND=tar -czf
#ARCHIVE=$(subst .tex,.tar.gz,$(MAIN))

add-dirs: $(FILES)
	cat $(FILES) | xargs -n1 dirname | sort -u | xargs svn add -N

dirs:
	cat $(FILES) | xargs -n1 dirname | sort -u | grep -Ev "^\\.$$"

add: add-dirs
	cat $(FILES) | xargs svn add -N

clean:
	-rm -f *.aux *.toc *.log *.bbl $(TARGET)

distclean: clean
	-rm -f $(FILES) $(DEPS)
	-rm $(BASE).zip $(BASE).tar.gz
	-rm -Rf $(BASE)

diff: $(FILES) $(TARGET)
	if [ ! -d "$(DIFFDIR)" ]; then \
		exit -1 ;\
	fi
	rm -Rf $(BASE).diff
	mkdir -p $(BASE).diff
	cd $(BASE).diff && \
	for d in `cat ../$(FILES) | xargs -n1 dirname | sort -u`;do \
		mkdir -p $$d; \
	done; \
	for f in `cat ../$(FILES) | grep -Ev .tex$$`;do \
		ln -sf `$(REALPATH) ../$$f` $$f; \
	done; \
	for f in `cat ../$(FILES) | grep -E .tex$$`;do \
		a=`$(REALPATH) ../$(DIFFDIR)/$$f` ; \
		b=`$(REALPATH) ../$$f` ; \
		if [ -f $$b ]; then \
			if [ -f $$a ]; then \
				latexdiff $$a $$b  > $$f; \
			else \
				ln -sf $$b $$f; \
			fi \
		fi \
	done; \

makediff: 
	if [ ! -d "$(BASE).diff" ]; then \
		exit -1 ;\
	fi
	cd $(BASE).diff && \
	make compile 



zip: $(ARCHIVE)

$(BASE).zip: zip

$(ARCHIVE): $(FILES) $(TARGET)
	mkdir -p $(BASE)
	rm -f $(ARCHIVE)
	cd $(BASE) && \
	for d in `cat ../$(FILES) | xargs -n1 dirname | sort -u`;do \
		mkdir -p $$d; \
	done; \
	for f in `cat ../$(FILES)`;do \
		ln -sf `$(REALPATH) ../$$f` $$f; \
	done;
	$(ARC_CMMAND) $@ $(BASE)
	rm -Rf $(BASE)

test: $(BASE).zip
	rm -Rf $(BASE)
	unzip $(BASE).zip
	cd $(BASE) && $(MAKE)

%.ebb: %.png
	ebb $<


force:

RSYNC_FLAG=-n

up: $(TARGET) $(BASE).zip
	rsync $(RSYNC_FLAG) --progress -av $(TARGET)  $(BASE).zip rsync://archive.arc.rice.edu/tafsm/PAPERS/$(BASE)/

extractbb:
	touch $@
