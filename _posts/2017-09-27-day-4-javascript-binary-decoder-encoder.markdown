---
layout: post
title:  "Day 4: JavaScript Binary Decoder/Encoder"
date:   2017-09-27 16:45:00 +0100
---
I love oneliners. I also love functional programming. I... would love Haskell, if I'd had the guts to learn it, but I love JavaScript. So after a year, I'm going to break the silence here with a simple little JavaScript oneliner... actually two.

#### Backstory

I received a contact request on LinkedIn by someone, who had this:

```
01110010 01100101 01100011 01110010 01110101 01101001 01110100 01100101 01110010
```

as her *role*.

Clever, she's trying to lure me into some coding, well no way lady! I could resist it for a week, but then a reminder came, that she is still waiting for my approval, and I saw it again. Binary numbers. FINE! I'll decode it, you persistent little...

#### Decoding

So, I have this:

```js
var role = "01110010 01100101 01100011 01110010 01110101 01101001 01110100 01100101 01110010";
```
It's one string, with eight-digit binary numbers separated by spaces and I want the decoded string as the result.

**Strategy:**

- split it at the spaces
- map through the numbers
  - convert them to integer
  - find the character represented by the integer
- join the characters again to one string

**Code:**
```js
role.split(" ")
    .map(item => String.fromCharCode(parseInt(item, 2)))
    .join("");
```

**Result:**

`recruiter` Who would have thought?

#### Encode

That was easy, so let's not stop here. What if I want to respond the same way? I have to turn my response

```js
var response = "I'm not interested.";
```

string into a string of eight-digit binary numbers.

**Strategy:**

- split the string
- map through each character
  - convert each character to an integer
  - convert the integers to binary numbers
  - left pad them with zeroes to be 8 digits long
- join the binary numbers with spaces

**Code:**
```js
response
    .split("")
    .map(item => ("0000000" + item.charCodeAt().toString(2))
        .split("").slice(-8).join(""))
    .join(" ")
```
I think the padding part may need some explanation. So, I want 8-digit binary numbers, but `.toString(2)` would give me the shortest possible binary string. For `1` it returns `1` and for `2` it returns `10`, you get it. Now I prepend each of them with 7 zeroes, so even the shortest number will be 8 digits. Then I split the string and slice the last 8 characters and rejoin them. Voilà!

**Result:**

```
01001001 00100111 01101101 00100000 01101110 01101111 01110100 00100000 01101001 01101110 01110100 01100101 01110010 01100101 01110011 01110100 01100101 01100100 00101110
```

There you go, lady! :)

#### Try it out!

I created a little CodePen so that you can play around with it. Enjoy!

<p data-height="265" data-theme-id="light" data-slug-hash="xXrZeg" data-default-tab="result" data-user="aorcsik" data-embed-version="2" data-pen-title="JavaScript Binary Decoder/Encoder" class="codepen">See the Pen <a href="https://codepen.io/aorcsik/pen/xXrZeg/">JavaScript Binary Decoder/Encoder</a> by Antal Orcsik (<a href="https://codepen.io/aorcsik">@aorcsik</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

---

**Update:** changed anonymous functions to ES6 arrow functions, because they are nice and you should also learn to use them
