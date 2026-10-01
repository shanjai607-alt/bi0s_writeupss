# Notepad - Writeup

## Description
The application reflects user input directly into a webpage and a headless browser visits the saved note. The goal is to execute JavaScript and trigger an alert() to receive the flag.

## Solution

First, I tested a basic XSS payload:

<img src=x onerror=alert(1)>

The application rejected it with:

"The word 'alert' is blacklisted."

So the word `alert` was being filtered.

Instead of writing `alert` directly, I constructed the string using JavaScript:

window['al'+'ert'](1)

Then I used an image with an `onerror` event to execute it:

<img src=x onerror="window['al'+'ert'](1)">

The `src=x` causes the image to fail loading, which triggers `onerror`.

The browser evaluates:

window['al'+'ert'](1)

as:

window['alert'](1)

This successfully triggers the alert dialog when the note is rendered by the browser.

## Payload

<img src=x onerror="window['al'+'ert'](1)">

## Flag
bi0s{5ur3ly_j4v4scrip7_c4nn0t_bu1ld_s7ring5_r1ght}