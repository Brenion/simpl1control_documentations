def has_duplicate_characters(text: str) -> bool

Return True if any character appears more than once in text.
return len(set(text)) != len(text)

Usage

python\nprint(has_duplicate_characters("python"))   # False\nprint(has_duplicate_characters("hello"))    # True\nprint(has_duplicate_characters(""))         # False\n```\n\nIt is case-sensitive, so `"Aa"` has no duplicates. For case-insensitive checking, use `text.lower()`.'