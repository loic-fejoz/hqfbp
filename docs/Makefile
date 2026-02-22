all: 2026-03-06-amsat-f.pdf

%.pdf: %.md
	npx @marp-team/marp-cli $< -o $@ --allow-local-files

clean:
	rm -f 2026-03-06-amsat-f.pdf