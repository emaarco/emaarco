# Template

The blocks behind every entry in this list.

## Book page

Saved as `books/<book-title-in-kebab-case>.md`:

```markdown
[← All books](README.md)

# TITLE

<a href="BOOK_URL"><img src="COVER_URL" alt="Cover of TITLE by AUTHORS" width="120"></a>

**AUTHORS**<br>
PUBLISHER · EDITION, YEAR

[Book page ↗](BOOK_URL)

## 💭 My take

OPINION

## 💡 What I took away

LEARNINGS
```

Covers come from Open Library (`https://covers.openlibrary.org/b/isbn/ISBN-M.jpg`) or the publisher's page. Without a reliable cover, `covers/placeholder.svg` stands in.

## Entry in the overview

Listed under its topic in [`README.md`](README.md):

```markdown
- **[TITLE](BOOK_PAGE.md)** · AUTHORS<br>
  FIRST_SENTENCE_OF_MY_TAKE
```
