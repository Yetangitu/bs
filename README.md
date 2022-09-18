# bs - a BookStack API CLI

Use:  `bs BS_OPTIONS <target> <endpoint> API_OPTIONS`

This is _bs_, a command line client for the BookStack wiki engine. It
enables easy command line access to all exported API functions and as
such can be used to automate content and user management.


## EXTERNAL DEPENDENCIES

    - bash
    - curl
    - jq


## CONFIGURATION

There are two ways to point bs at the required url and authentication token:

1:  Run `bs config` or manually create a configuration file in the XDG
    configuration directory (`$HOME/.config/bs.conf`) with the following
    contents:

    url="https://bookstack.example.org"
    token="b0ada1c9e9a0057db44ac6dd684b93a4:8dde5bc903d297319abba326637af5f9"

    This is only an example, replace the values with those relevant
    to your BookStack instance and account.

    In addition to the default url=/token= parameters bs also supports
    named hosts and users which can be selected on invocation, these
    are configured by adding _hostname/_username to the variable name:

    url_host1="https://host1.example.org"
    url_host2="https://anotherhost.example.org"

    token_librarian="...:..."
    token_guest="...:..."

2:  Create environment variables with the required info, e.g.

    $ BOOKSTACK_URL="https://bookstack.example.org"
    $ BOOKSTACK_TOKEN="b0ada1c9e9a0057db44ac6dd684b93a4:8dde5bc903d297319abba326637af5f9"
    $ export BOOKSTACK_URL BOOKSTACK_TOKEN

Environment variables take precedence over the configuration file, this makes
it possible to temporarily use a different instance or account:

    $ BOOKSTACK_TOKEN="...:..." bs books list

## BS_OPTIONS

As stated above it is possible to add named hosts and users in the
configuration file. Use the following options to select one of these:

    -H HOSTNAME     use the host configured as url_HOSTNAME in bs.conf
    -U USERNAME     use the token configured as token_USERNAME in bs.conf
    -h              show this help message (also: bs help, bs -help, bs --help)

These options have to come directly after the program name:

    bs -H local_library -U librarian books list


## API ENDPOINTS

These are the BookStack API [1] targets and endpoints as they are
supported in _bs_:

### pages, chapters, books

    list, create, read, update, delete, export-html, export-pdf,
    export-plain-text, export-markdown

### shelves, attachments, users

    list, create, read, update, delete

### docs

    display, json

### search

    all (implicit, just use 'bs search -q <searchterm>')

### recycle-bin

    list, restore, destroy


## API_OPTIONS

API parameters are passed as options, check the API documentation [1]
to learn which parameters are available. Use these options for:

    list:

    -c COUNT    How many records will be returned in the response.
    -o OFFSET   How many records to skip over in the response. 
    -s SORT     Which field to sort on, +/- prefix indicates sort
                direction.
    -f FILTER   Filter to apply to query, consult API documentation
                for filter syntax.

    create, read, update, delete:

    -i ID       Record ID to read/update/delete
    -n NAME
    -d DESCRIPTION
    -t TAGS     Tags, use 'tag1name=tag 2 value;tag2name=tag 2 value;...'
    -I IMAGE    Filename for image to add

    parameters specific to chapters and pages:

    -b BOOK_ID

    parameters specific to pages:

    -c CHAPTER_ID
    -h HTML     HTML text to add, mutually exclusive with -m
                use @/path/to/file.html to read text from a file

    -m MARKDOWN Markdown text to add, mutually exclusive with -h
                use @/path/to/file.md to read text from a file

    parameters specific to attachments:

    -u UPLOADED_TO
    -f FILE     Filename to attach, mutually exclusive with -l
    -l LINK     Link to attach, mutually exclusive with -f

    parameters specific to shelves:

    -B BOOKS    Book IDs for shelf, use '1,2,3...'

    parameters specific to users:

    -e EMAIL
    -a EXTERNAL_AUTH_ID
    -L LANGUAGE
    -p PASSWORD
    -r ROLES    Roles, use '1,2,3...'
    -s          Send invite
    -M MIGRATE_OWNERSHIP_ID Migrate data ownership for deleted user.

    search:

    -q QUERY    Search query
    -p PAGE     Results page to show
    -c COUNT    Number of results per page
                Count value is taken as a suggestion only,
                see API documentation [1]


## OUTPUT

All API responses are in compact _json_ format. To make this a bit more human readable the
output can be piped through _jq_:

    bs books list|jq '.'


## EXAMPLES

List available books

    bs books list

Again, this time 500 results, start at offset #1000

    bs books list -c 500 -o 1000
        
Show details for page with id 12345

    bs pages read -i 12345

Create a book with title "Book of Books" and description "A book about
books" with a cover image of a Pozzebok and two tags

    bs books create -n "Book of Books" -d "A book about books" \
    -I /tmp/pozzebok.png -t 'subject=a book on books;cover=image of a pozzebok'

Search for this book

    bs search -q "Book of Books"

Delete the book with ID=23456

    bs books delete -i 23456

Create a chapter in book #4242

    bs chapters create -b 4242 -n "First Chapter" -d "Obviously the first chapter"

Create page and attach it to chapter #456, using markdown. Notice the
markdown content is spread over several lines with empty lines in between
and that double quotes in the content are escaped.

    bs pages create -c 456 -n "Page of Pages" -m "#Page of Pages
    This is the first line of the first page.

    And this is the second one, a _masterwork_ in the making.

    Don't forget to \"escape\" double quotes"

Delete that useless page you just made

    bs pages delete -i 66753

Create another page, this time reading markdown/html page content from a file

    bs pages create -c 456 -n "Page the second" -m @/home/me/Documents/magnum_opus.md
    bs pages create -c 456 -n "Page the second" -h @/home/me/Documents/magnum_opus.html

Export a book/chapter/page to PDF/HTML/Markdown, redirecting output to a file

    bs books export-pdf -i 6679 > book_6679.pdf
    bs chapters export-html -i 112 > chapter_112.html
    bs page export-markdown -i 5665 > page_5665.md

Attach a file to a page with ID 5678

    bs attachments create -n "Deep Image" -u 5678 -f /tmp/deepimage.png

Attach a link to the same page

    bs attachments create -n "Deeper Image" -u 5678 -l 'https://example.org/deeperimage.png'

List the first 200 attachments

    bs attachments list -c 200

Create a shelf and put a few books on it

    bs shelves create -n "Billy" -d "IKEA Bookshelf" -B 456,654,345,3343 -I /tmp/billy.jpg

List the contents of the recycle bin, showing 100 entries

    bs recycle-bin list -c 100

Restore entry #34 from the recycle bin

    bs recycle-bin restore -i 34

Create a user

    bs users create -n "Billy Bob" -e "billybob@example.org" -L "Klingon" -p "b1llyb0b123"

List the first 500 books on host `remote_library` as seen by user `guest`

    bs -H remote_library -U guest books list -c 500

Edit the _bs_ config file

    bs config

---

    [1] https://demo.bookstackapp.com/api/docs

