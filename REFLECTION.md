## Q1. 

The decision is at src/scanner.rs:126. 

if self.peek() == '.' && self.peek_next().is_ascii_digit() decides whether a number is a fractional belongs in a number by assessing the dot (decimal). It decides whether a . begins a fractional part rather than being a stray character.

For the input `5.`, my scanner holds first and fails after. 
Nothing follows the `.`, so it ends as a `5` and the dot remains unconsumed. 
The next scan_token call reads the `.`. 
No arm matches it, so the catch-all records [line 1] Error: Character is not part of any token.

A fraction needs one or more digits after the dot, and Kobo has no `.` token, so a stray dot becomes an error.

## Q2. 

I change the line counter at src/scanner.rs:63.

It runs each time scan_token reads a newline character as the start of a new token. [when does this one run?]
The code at src/scanner.rs:105 runs once for each newline character that appears between the opening " and the closing ". 
A string with no line break in it never reaches that line. 

It differs from src/scanner.rs:63 because line 63 is in `scan_token`. It runs when a newline is the start of a new token, so it covers newlines between tokens.
Line 105 is in `string()`. Once string() starts, its own loop consumes every character up to the closing quote with self.advance(). scan_token isn't called during that time, so it never sees those newlines, and line 63 can't count them. 

Without line 105, a multi-line string would leave the counter behind, and every token after it would be reported on too early a line. 

For a file that ends in two blank lines, my EOF token carries line 1. src/scanner.rs:36 reads the line of the last token in self.tokens (self.tokens.last()). If there are no tokens, it falls back to 1, as spec 6.1 says for empty or comment-only files. 

It does not use self.line, the live counter. By the end of that file the counter has passed three newlines, so it would be 4, which points at nothing real. It would push EOF onto a later line. Two files with identical code, one with a trailing newline and one without, would then print different token streams, so a newline added by an editor would change the output. Spec 6.1 forbids that, and it is the same reason spec 5 reports at end parse errors at the last real token.

## Q3. 
The line was wrong in commit [hash: 9db56ba]:
    102:            if self.peek() == '"' {

I fixed it in commit [hash: 7887ec5]:
    102:            if self.peek() == '\n' {

It sat inside a loop that only ran while peek() is not ". A character can't be both.
`self.line += 1` underneath it could never run, and newlines inside strings were never counted.