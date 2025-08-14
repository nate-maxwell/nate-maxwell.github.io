---
layout: post
title: "PY Poor Man's IDE"
date: 2023-03-22
permalink: /py-poor-mans-ide/
author: Nate Maxwell
categories:
    - "Python"
---

Lately I’ve found myself fascinated with IDE development. I was recently inspired by seeing a github repo where someone had used PyQt and Scintilla to parse various languages in a custom developer environment. I wanted to try this myself, although in a much simpler approach, mainly to learn about syntax highlighting. I didn’t want to get lost in learning about massive parsing and lexing frameworks, like Scintilla, but I was curious about state-machine or rules based syntax highlighting.

Admittedly I got kinda bored partway through, but I did manage to make a working project. There isn’t much to go in depth with, but it does let you edit and save files. It's missing various basic features but I don’t think I will continue working on it. Anyways, here is the Cypher IDE.

<img src="https://camo.githubusercontent.com/29e99033c10b6bb16cbdbd73636532a133ecf466be45b9aad4918fe6f26804d9/68747470733a2f2f692e696d6775722e636f6d2f664c666c4355362e706e67">

Again, this was mainly to learn more about syntax highlighting and here is the basics of how Cypher works:

In `./cypher/languages/` there is `json_syntax.py` and `python_syntax.py`. Put simply, we define groups of characters that get certain text highlights, and then load the syntax file based on the loaded file suffix.

This is primarily done via the following function:

```python
def color_format(color: str, style: str = '') -> QtGui.QTextCharFormat:
    _color = QtGui.QColor()
    _color.setNamedColor(color)

    _format = QtGui.QTextCharFormat()
    _format.setForeground(_color)
    if 'bold' in style:
        _format.setFontWeight(QtGui.QFont.Bold)
    if 'italic' in style:
        _format.setFontItalic(True)

    return _format
```

From here we can start grouping syntax concepts by colors

```python
STYLES = {
    'keyword': color_format('cyan'),
    'operator': color_format('white'),
    'brace': color_format('orange'),
    'defclass': color_format('lightGreen'),
    'string': color_format('grey'),
    'string2': color_format('grey'),
    'comment': color_format('darkGreen', 'italic'),
    'self': color_format('orange'),
    'numbers': color_format('magenta'),
}
```

We can then group symbols together as keywords, operators, and braces (when working with python).

To my surprise, Qt comes with a fair amount of text formatting utilities specifically for syntax use. Here I have extended the `QtGui.QSyntaxHighlither` class.
It uses regex to define rules, and these rules determine when and when not to apply our `STYLES` dict. Some challenges were getting string highlighting to work for multi line strings,
as we parse each line in a code block out individually and it could contain string forms of example code.

```python
class PythonHighlighter(QtGui.QSyntaxHighlighter):
    """Syntax highlighter for the Python language."""
    # Python keywords
    keywords = [
        'and', 'assert', 'break', 'class', 'continue', 'def',
        'del', 'elif', 'else', 'except', 'exec', 'finally',
        'for', 'from', 'global', 'if', 'import', 'in',
        'is', 'lambda', 'not', 'or', 'pass', 'print',
        'raise', 'return', 'try', 'while', 'yield',
        'None', 'True', 'False',
    ]

    # Python operators
    operators = [
        '=',
        # Comparison
        '==', '!=', '<', '<=', '>', '>=',
        # Arithmetic
        '\+', '-', '\*', '/', '//', '\%', '\*\*',
        # In-place
        '\+=', '-=', '\*=', '/=', '\%=',
        # Bitwise
        '\^', '\|', '\&', '\~', '>>', '<<',
    ]

    # Python braces
    braces = [
        '\{', '\}', '\(', '\)', '\[', '\]',
    ]

    def __init__(self, parent: QtGui.QTextDocument = None) -> None:
        super().__init__(parent)

        # Multi-line strings (expression, flag, style)
        self.tri_single = (QtCore.QRegExp("'''"), 1, STYLES['string2'])
        self.tri_double = (QtCore.QRegExp('"""'), 2, STYLES['string2'])

        rules = []

        # Keyword, operator, and brace rules
        rules += [(r'\b%s\b' % w, 0, STYLES['keyword']) for w in PythonHighlighter.keywords]
        rules += [(r'%s' % o, 0, STYLES['operator']) for o in PythonHighlighter.operators]
        rules += [(r'%s' % b, 0, STYLES['brace']) for b in PythonHighlighter.braces]

        # All other rules
        rules += [
            # 'self'
            (r'\bself\b', 0, STYLES['self']),

            # 'def' followed by an identifier
            (r'\bdef\b\s*(\w+)', 1, STYLES['defclass']),
            # 'class' followed by an identifier
            (r'\bclass\b\s*(\w+)', 1, STYLES['defclass']),

            # Numeric literals
            (r'\b[+-]?[0-9]+[lL]?\b', 0, STYLES['numbers']),
            (r'\b[+-]?0[xX][0-9A-Fa-f]+[lL]?\b', 0, STYLES['numbers']),
            (r'\b[+-]?[0-9]+(?:\.[0-9]+)?(?:[eE][+-]?[0-9]+)?\b', 0, STYLES['numbers']),

            # Double-quoted string, possibly containing escape sequences
            (r'"[^"\\]*(\\.[^"\\]*)*"', 0, STYLES['string']),
            # Single-quoted string, possibly containing escape sequences
            (r"'[^'\\]*(\\.[^'\\]*)*'", 0, STYLES['string']),

            # From '#' until a newline
            (r'#[^\n]*', 0, STYLES['comment']),
        ]

        # Build a QRegExp for each pattern
        self.rules = [(QtCore.QRegExp(pat), index, fmt)
                      for (pat, index, fmt) in rules]

    def highlightBlock(self, text: str):
        """
        # Override
        Apply syntax highlighting to the given block of text.
        Args:
            text(str): The text to analyze for highlighting.
        """
        self.tripleQuoutesWithinStrings = []
        # Do other syntax formatting
        for expression, nth, format in self.rules:
            index = expression.indexIn(text, 0)
            if index >= 0:
                # if there is a string we check
                # if there are some triple quotes within the string
                # they will be ignored if they are matched again
                if expression.pattern() in [r'"[^"\\]*(\\.[^"\\]*)*"', r"'[^'\\]*(\\.[^'\\]*)*'"]:
                    innerIndex = self.tri_single[0].indexIn(text, index + 1)
                    if innerIndex == -1:
                        innerIndex = self.tri_double[0].indexIn(text, index + 1)

                    if innerIndex != -1:
                        tripleQuoteIndexes = range(innerIndex, innerIndex + 3)
                        self.tripleQuoutesWithinStrings.extend(tripleQuoteIndexes)

            while index >= 0:
                # skipping triple quotes within strings
                if index in self.tripleQuoutesWithinStrings:
                    index += 1
                    expression.indexIn(text, index)
                    continue

                # We actually want the index of the nth match
                index = expression.pos(nth)
                length = len(expression.cap(nth))
                self.setFormat(index, length, format)
                index = expression.indexIn(text, index + length)

        self.setCurrentBlockState(0)

        # Do multi-line strings
        in_multiline = self.match_multiline(text, *self.tri_single)
        if not in_multiline:
            in_multiline = self.match_multiline(text, *self.tri_double)

    def match_multiline(self, text: str, delimiter: QtCore.QRegExp,in_state: int, style: QtGui.QColor) -> bool:
        """
        Do highlight of multi-line strings.

        Args:
            text(str): The text to perform the highlight matching on.
            delimiter(QRegExp): A QRegExp for triple-single-quotes and triple-double-quotes
            in_state(int): A unique integer to represent the corresponding state changes
             when inside those strings.
            style(QtGui.QColor): Which color to set the multiline text to.
        Returns:
            bool: Returns True if we're still inside a multi-line string when this function
             is finished.
        """
        # If inside triple-single quotes, start at 0
        if self.previousBlockState() == in_state:
            start = 0
            add = 0
        # Otherwise, look for the delimiter on this line
        else:
            start = delimiter.indexIn(text)
            # skipping triple quotes within strings
            if start in self.tripleQuoutesWithinStrings:
                return False
            # Move past this match
            add = delimiter.matchedLength()

        # As long as there's a delimiter match on this line...
        while start >= 0:
            # Look for the ending delimiter
            end = delimiter.indexIn(text, start + add)
            # Ending delimiter on this line?
            if end >= add:
                length = end - start + add + delimiter.matchedLength()
                self.setCurrentBlockState(0)
            # No; multi-line string
            else:
                self.setCurrentBlockState(in_state)
                length = len(text) - start + add
            # Apply formatting
            self.setFormat(start, length, style)
            # Look for the next match
            start = delimiter.indexIn(text, start + length)

        # Return True if still inside a multi-line string, False otherwise
        if self.currentBlockState() == in_state:
            return True
        else:
            return False
```

And there is the primary goal I set out to do when making this editor. Feel free to read through the line comments to get an idea of the control flow in each section.
