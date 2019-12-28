# You must have jekyll and R installed, and they must be in PATH. For R, you
# need to install the servr package.

all:
	Rscript -e "servr::jekyll('..', command='bundle exec jekyll build')"
	#Rscript -e "servr::jekyll('..')"
	#Rscript -e "servr::jekyll('..', serve=FALSE)"

clean:
	rm -r ../_site
