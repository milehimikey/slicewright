MARP := npx -y @marp-team/marp-cli

DECKS := teaser-5min full-walkthrough-25min

.PHONY: all html pdf clean

all: html pdf

html:
	@for d in $(DECKS); do \
		$(MARP) $$d.md -o $$d.html; \
	done

pdf:
	@for d in $(DECKS); do \
		$(MARP) $$d.md -o $$d.pdf --allow-local-files; \
	done

clean:
	rm -f $(DECKS:%=%.html) $(DECKS:%=%.pdf)
