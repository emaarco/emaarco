# Book template

Adding a book takes two steps: a page for the book and a cover in the overview.

- `BOOK_URL`: a freely accessible book or publisher page.
- `COVER_URL`: a stable cover image, e.g. `https://covers.openlibrary.org/b/isbn/ISBN-M.jpg`. Without a reliable cover, use `covers/placeholder.svg`.
- `CATEGORY`: one of the sections in the overview, with its emoji.

## 1. Book page

Save it as `books/<book-title-in-kebab-case>.md`:

```markdown
[← All books](README.md)

# TITLE

<table>
<tr>
<td width="160" valign="top"><a href="BOOK_URL"><img src="COVER_URL" alt="Cover of TITLE by AUTHORS" width="140"></a></td>
<td valign="top">
<b>AUTHORS</b><br>
PUBLISHER · EDITION, YEAR<br><br>
CATEGORY<br><br>
<a href="BOOK_URL">Book page ↗</a>
</td>
</tr>
</table>

## 💭 My take

OPINION

## 💡 What I took away

LEARNINGS
```

## 2. Cover in the overview

Add a cell to the matching section of [`README.md`](README.md). Each row holds four cells, so start a new `<tr>` when a row is full:

```html
<td align="center" valign="top" width="25%">
<a href="BOOK_PAGE.md"><img src="COVER_URL" alt="Cover of TITLE by AUTHORS" height="160"></a><br>
<sub><b>SHORT_TITLE</b><br>AUTHORS</sub>
</td>
```
