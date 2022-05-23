# Use gitattributes to control files' line endings

Use [`.gitattributes`](https://git-scm.com/docs/gitattributes)
to control the line endings for files that are in the repository.

The above settings are useful when working with others who use other Operating Systems, because
each OS has its own line endings for files.

When determining which settings to apply, Git gives higher precedence to the `.gitattributes` file.

- Every text file is checked out with `LF` line endings in the working directory (`eol=lf`)
- Every potentially introduced `CRLF` in a text file will be converted back to `LF` on staging (`* text=auto`)

[Git documentation](https://www.git-scm.com/book/en/v2/Customizing-Git-Git-Configuration) describes
the "Formatting and Whitespace" options available when installing it.
But I'm not going to check each everyone's git installation to confirm that they've set these up as I'd like

After coming across these settings in a [spectral's commit](https://github.com/stoplightio/spectral/commit/5459fe1fc7ff903b8702e8f5c0690d6d3b7c9518),
I decided to use them too and add them to my [git utilities](https://github.com/juliusgb/utils/blob/main/git/.gitattributes).
