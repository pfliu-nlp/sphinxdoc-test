# Publishing sphinx-generated docs on github

References:<br> 
1. https://daler.github.io/sphinxdoc-test/includeme.html
2. https://www.sphinx-doc.org/en/master/tutorial/getting-started.html

## Protocal
## Set up main repository
First set up your main repo. These are the commands I used to set up this very repo (how meta!):
```
mkdir sphinxdoc-test
cd sphinxdoc-test
git init
touch README
git add README
git commit -m 'first commit'
git remote add origin git@github.com:daler/sphinxdoc-test.git
git push origin master
```
Throughout this document, I’ll refer to this as the ‘main repo’ or the ‘code dir’.


## Set up sphinx within main repository
Make a dir, `docs`, that will store documentation source from Sphinx:
```
mkdir sphinxdoc-test/docs
```
Then set up Sphinx from the docs dir, accepting all the defaults as you see fit:
```
cd docs
sphinx-quickstart
```

## Set up separate docs repository
Now we need to set up a completely new directory that will serve as the build directory for Sphinx. Here I’m calling it `sphinxdoc-test-docs`. Note that it’s outside of the main repo dir:

```
cd ..
mkdir sphinxdoc-test-docs
cd sphinxdoc-test-docs
```

Then clone the repo you just set up on github into a dir called `html` (which will be created automatically with the following command):
```
git clone git@github.com:daler/sphinxdoc-test.git html
cd html
```
The `html` dir now has a clone of the repo.

Next, create a new branch called `gh-pages`. This is a special branch name that github looks for in order to build static html pages:
```
git branch gh-pages
```

The following commands do git fancy stuff that I don’t completely understand yet, suffice to say that after these 3 commands you switch to the new branch gh-pages and the branch is cleaned out with no files in it:
```
git symbolic-ref HEAD refs/heads/gh-pages  # auto-switches branches to gh-pages
rm .git/index
git clean -fdx
```

And confirm we’re on gh-pages:
```
git branch
```

I'll refer to this as `gh-pages repo`.

## Makefile changes
OK, now the docs repo is set up. Now it’s time to make some changes to the sphinx-generated Makefile back in the main repo so that it builds documentation in our new gh-pages branch and directory, instead of cluttering the main code dir.

So go back to the code dir’s `doc` dir:
```
cd ../sphinxdoc-test
cd docs
```

Here are the changes we're going to make to `sphinxdoc-test/docs/Makefile`, first, change:
```
BUILDDIR = build
```
to
```
BUILDDIR = ../../sphinxdoc-test-docs
```
The first new line points to the new dir and gh-pages branch we just set up. So now, running `make html` in `sphinxdoc-test/docs` will create an html dir in `../../sphinxdoc-test-docs` . . . and luckily, that’s exactly what we set up the gh-pages repo in.

## index.rst changes
Next, I’d like to only worry about making changes in a single file (`README.rst`), and have that propagated to all the docs in various places. On github, if you have a `README.rst` file in the root dir, it’ll be converted to nice-ish looking docs. (Sphinx is much better looking, plus can include module, class, and function documentation to boot, hence going through all this trouble).

So we need to point sphinx’s `index.rst` to the `README.rst` file in the root of the main repo. Turns out that relative path names don’t work in index.rst, so here’s a workaround:

Make a new file, `sphinxdoc-test/docs/source/includeme.rst`. In there, put an include directive pointing to the true `README.rst`. So `includeme.rst` should look like this:

```
.. include:: ../../README.rst
```
Then in `index.rst`, add `includeme` to the toctree. So the relevant part of `index.rst` should look something like:
```
.. toctree::
   :maxdepth: 2

   includeme
```
OK, we should be done with the setup now.

## Initial creation and commit workflow
Commit all code and `README.rst` (and any other doc source files) in the main repo, like always:
```
git add docs
git add README.rst
git commit -m "added docs and README.rst"
```

Then, when you're ready to recreate the docs.
```
cd docs
make html
```

Next, change to the gh-pages repo dir and commit the stuff that the `make html` command made:
```
cd ../sphinxdoc-test-docs/html
git add .
git commit -m "rebuilt docs"
```

And then publish the newly built docs:
```
git push origin gh-pages
```
Anyway, now you can view your new pages on `http://<user>.github.com/<repo>`. So in this case, it’s http://daler.github.com/sphinxdoc-test.


## Add a .nojekyll file
The last thing we have to do is add an empty file called `.nojekyll` in the docs repo. This tells github’s default parsing software to ignore the sphinx-generated pages that are in the gh-pages branch. Make sure you commit it, too:

```
cd sphinxdoc-test-docs/html
touch .nojekyll
git add .nojekyll
git commit -m "added .nojekyll"
```

## Directory structure
So that we’re on the same page, the final directory structure looks like this:
```
sphinxdoc-test
|-- pymodule              <-- whatever your normal python package dir structure is
|   |-- somepythonmodule.py
|   `-- othercode.py
|-- docs
|   |-- Makefile          <-- edited as described above
|   `-- source
|       |-- conf.py
|       |-- includeme.rst <-- edited as described above
|       `-- index.rst     <-- edited as described above
|-- manual.pdf            <-- created by running make latexpdf
`-- README.rst            <-- where you do most of your writing

sphinxdoc-test-docs
|-- doctrees              <-- this dir is autogenerated, but not
|   |-- environment.pickle     commited to gh-pages
|   |-- includeme.doctree
|   |-- index.doctree
|   `-- README.doctree
`-- html                  <-- The docs repo, on the gh-pages branch.
    |-- genindex.html          Everything under here is committed.
    |-- includeme.html
    |-- index.html
    |-- objects.inv
    |-- README.html
    |-- search.html
    |-- searchindex.js
    |-- _sources
    |   |-- includeme.txt
    |   |-- index.txt
    |   `-- README.txt
    `-- _static
        |-- basic.css
        |-- default.css
        |-- doctools.js
        |-- file.png
        |-- jquery.js
        |-- minus.png
        |-- plus.png
        |-- pygments.css
        |-- searchtools.js
        |-- sidebar.js
        `-- underscore.js

```
