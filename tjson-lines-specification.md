Text Json TJSON Lines specification (STUB: will be a full specification later)

# NOTE ON TJSON LINES AND WHERE THIS IS GOING - PLEASE READ

TLDR: Use BOLD MULTILINES only, two EOL after each TJSON value

I'm going to extend this more later, but I wanted to put this up quickly so that everyone knows conceptually where this is going before coming up with their own inconsistent rules without any guidance whatsoever.

This is basically JSON Lines extended to TJSON.

In JSONL, we take something away from JSON - (interior unescaped EOL), and add something, (EOL after each object).

In TJSON lines, we are doing the same thing - we take away interior EOL EOL, and the way we do this is by simply not generating TRANSPARENT multiline strings that contain their own empty lines, falling back to some other form.  It's probably a good idea for readability to take away MINIMAL for the same reason, but minimal is guaranteed to parse because an interior empty line will be EOL (n spaces) EOL, and you are guaranteed at least a n of two internally, so MINIMAL will still parse but probably ought to be avoided for aesthetic reasons.  BOLD is the preferred format for logging anyway, so I would strongly suggest that you use BOLD always for TJSON Lines, even though the others should parse without issues in most circumstances.

In TJSON Lines - when you see EOL EOL, you know you are done with the TJSON object.

When generating TJSON Lines, we simply add extra EOL to the end, so each output ends in (anything that's not an EOL) (EOL) (EOL).