
# Table of Contents

1.  [Doom Emacs](#org9cde933)
    1.  [org-agenda](#org7799958)
    2.  [pdf-tools](#org53009b1)
    3.  [Quick Functions to sort](#org6789267)
2.  [HPC Usage](#org6b3bba2)
3.  [Conda](#orgf776080)
4.  [Poetry](#orgc185a62)
    1.  [Install](#org2db652f)
    2.  [Development](#org762fcec)
    3.  [Uninstall](#org531cbd2)
5.  [Pyenv](#org466b3fe)
6.  [Python virtual environments](#orgd8267e5)
7.  [PostgreSQL](#org5a4fd69)
    1.  [PSQL commands](#org5f243a0)
    2.  [SQL commands](#org326f6f5)
        1.  [Solving foreign key issues](#orga58490c)
8.  [Apptainer](#org0ad3200)

Basic instructions for a number of software(s) and packages that aren&rsquo;t specific to an instrument, etc.


<a id="org9cde933"></a>

# Doom Emacs


<a id="org7799958"></a>

## org-agenda

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left">SPC o A</td>
<td class="org-left">open the org agenda menu</td>
</tr>

<tr>
<td class="org-left">SPC m t</td>
<td class="org-left">change the todo state of a task</td>
</tr>

<tr>
<td class="org-left">SPC s i</td>
<td class="org-left">search for headings</td>
</tr>

<tr>
<td class="org-left">C-c C-j</td>
<td class="org-left">org-goto -&gt; find by headings</td>
</tr>

<tr>
<td class="org-left">C-c C-f</td>
<td class="org-left">go to next heading same level</td>
</tr>

<tr>
<td class="org-left">C-c C-b</td>
<td class="org-left">go to previous heading same level</td>
</tr>

<tr>
<td class="org-left">C-c C-t</td>
<td class="org-left">change state of org todo</td>
</tr>
</tbody>
</table>


<a id="org53009b1"></a>

## pdf-tools

Setting up tools so that PDFs can be viewed on macOS.
First install requirements.

    brew install expat
    
    ## add to ~/.zshrc
    export PKG_CONFIG_PATH="/opt/homebrew/lib/pkgconfig:/opt/homebrew/opt/expat/lib/pkgconfig:$PKG_CONFIG_PATH"

Follow instructions from there.

    M-x install-packages
        > pdf-tools
    
    M-x pdf-tools-setup

If you try to run these steps without installing expat, before the expat error comes up there will be an error about &rsquo;poppler&rsquo; not being installed/at the correct version, even if this is not true. This can be safely ignored and installation should run fine once dependencies are met.


<a id="org6789267"></a>

## Quick Functions to sort

elfeed (C-x w) - open elfeed
    henne/<elfeed-get-arxiv> - elfeed save selected file

citar-open - open citar
citar-insert-citation - add a citation to the current file
citar-add-file-to-library - (have to press a button first) then select option used to add missing pdf

org-ref-set-bibtex-keywords - add keywords to file

arxiv-add-bibtex-entry - manually add bibtex from arxiv doi if it&rsquo;s not downloading (check to see if added to bib already)
arxiv-get-pdf-add-bibtex-entry - manually add a file to bib and download pdf from arxiv number

doi-utils-get-bibtex-entry-pdf - get a pdf for an entry if it doesn&rsquo;t exist
doi-utils-add-bibtex-entry-from-doi - add doi entry to bib file (should also download pdf)


<a id="org6b3bba2"></a>

# HPC Usage

Submit a file to the job queue.
`sbatch file.slm`

Check all jobs statuses for username.
`squeue -u ah724`

Delete a file from the queue.
`sdel <jobid>`


<a id="orgf776080"></a>

# Conda

Create an environment:
`conda create -n <name> python=<x.y>`

List all environments:
`conda info -e`

Activate/deactivate an environment:
`conda activate <envname>`
`conda deactivate`

Delete an environment:
`conda remove -n <env_name> --all`

List all packages and versions in an environment:
`conda list`

Update a package.
`conda update <package>`


<a id="orgc185a62"></a>

# Poetry

Used to manage, develop and publihs code repositories. Install to local machine, and it can be used to create it&rsquo;s own virtual environment to install dependencies and develop code in. Don&rsquo;t use it from within a conda environment.


<a id="org2db652f"></a>

## Install

Install poetry command:
`curl -sSl https://install.python.poetry.org | python3`

Possibly optional: on my machine, poetry wasn&rsquo;t able to create it&rsquo;s own directory automatically due to admin permissions, so made my own directory and pointed poetry to it - add these lines to `/.zprofile:
~export POETRY_CONFIG_DIR=/Users/ah724/.poetry/`
`export PATH=/Users/ah724/.local/bin:$PATH`

Setup the link to PyPi - aquire token from website first.

    # Add repository link.
    poetry config repositories.<package_name> https://upload.pypi.org/legacy/
    
    # Add user credentials.
    poetry config http-basic.<package_name> __token__ pypi-<token>
    
    # Check to see if it's worked.
    poetry config repositories


<a id="org762fcec"></a>

## Development

With development directory open in VSCode, Emacs, etc.

    poetry init # init a new project
    poetry shell # open the poetry virtual shell
    poetry install # install dependencies
    poetry build # build package
    poetry add <package_name> # add package dependencies
    poetry add --group dev <package_name> # add development package dependencies
    
    # Build and publihs to PyPi repository.
    poetry publish --build --repository <pypi_pkg_name>


<a id="org531cbd2"></a>

## Uninstall

To uninstall:
~curl -sSL <https://install.python-poetry.org> &#x2013;uninstall | python3


<a id="org466b3fe"></a>

# Pyenv

Manager to install and swap between multiple versions of python.

`pyenv install 3.X.X` -> install python version

`pyenv global 3.X.X` -> make this version the global

`pyenv shell <version>` -> activate this version for just the shell

`pyenv versions` -> list installed versions

Python2 -> You can use 2.7.18


<a id="orgd8267e5"></a>

# Python virtual environments

Use the command:
`python -m venv <path>`


<a id="org5a4fd69"></a>

# PostgreSQL

Database application for SQL tables.


<a id="org5f243a0"></a>

## PSQL commands

    # connect to sql server at host -h with database -d with username -U
    \psql -U ahhennessey -h vlo.science.uva.nl -d <databasename>
    
    # list databases
    \list
    \l
    
    # show tables within databases
    \dt
    
    # connect or change database
    \connect <table>
    \c <table>
    
    # copy table to filepath as csv
    \copy extractedsource TO '/scratch/ahennessey/csvs/extract_240418a.csv' CSV HEADER;
    
    # create a database server
    initdb trap_db
    
    ## (not sure about the commands below)
    # connect to server
    pg_ctl -D ./trap_db/ -l logfile start-
    
    # create/remove database
    createdb <name>
    dropdb <name>
    
    # access psql command line
    psql <dbname>


<a id="org326f6f5"></a>

## SQL commands

Use `psql` to enter the command prompt, then you can use SQL commands:

    -- show contents of a table
    SELECT * FROM <table>;
    
    -- delete all rows from table; delete by condition
    DELETE FROM <table>;
    DELETE FROM <table> WHERE <column_name>='<string>'
    
    -- delete the table completely
    DROP TABLE <table>;


<a id="orga58490c"></a>

### Solving foreign key issues

    -- remove and then add foreign key issues
    ALTER TABLE <table> DROP CONSTRAINT <contraint_name>;
    ALTER TABLE <table> ADD CONSTRAINT <constraint_name> FOREIGN KEY (<col_id>) REFERENCES table(<col_id>);


<a id="org0ad3200"></a>

# Apptainer

Replacement of Singularity. For all the (few) times I&rsquo;ve had to use it so far, any command for Singularity still works if you just replace it with &rsquo;apptainer&rsquo;.

To open the container into a shell:
`singularity shell <container.sif>`

More usefully you&rsquo;ll most likely need to bind scratch (and maybe data) otherwise the container cannot &rsquo;see&rsquo; them.
This also needs to be done if you&rsquo;re just running the container, like as part of a HPC script.
`singularity shell --bind /scratch:/scratch --bind /data:/data <container.sif>`

