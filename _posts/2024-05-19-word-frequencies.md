---
title: 'Learning a foreign language with Anki'
subtitle: "finding the most common words"
author:
- Rafal Czyzewski
categories: python, spanish
tags: productivity 
description: |
  There is a lot of vocabulary to learn in any language, even natives don't know them all. 
  I decided to enrich my Anki decs with word frequency - so I can focus on the words that I have higher chanes to 
  encounter and as a consequence a lower chances to forget.
---
# Using Anki

As the first post in a blog is the hardest, let's take it easy and write a few sentences about
using `Anki` - https://apps.ankiweb.net/

In a nutshell: you have a deck of cards: with question in the front and the answer in the back. You pick a card from the
top of the deck. Remind yourself the answer, confront with the answer from the back of the card. Evaluate your answer.
The better/faster you recall the right answer, the longer might be the period for the next reminder.
So the idea is super simple, moreover https://ankiweb.net/shared/decks provide many decs that might be easy for the
start.

However, I have found that using a decs that someone already has prepared for me just doesn't work, at least for me.
When looking at words in a different language, they many seems to be just a collection of a random characters - at first
sight it's not a big difference between those words, and telephone numbers.
In order to avoid such situations I'm creating my own decks wile reading a books - so I know the context of the word.

As I'm a developer, I have a set of scripts that are collecting vocabluary from kindle. Then there is a web interface to
enrich those words with transalationn and usage from online resources.
This way I can decide if the word is not a mistake, translation is clear, examples are vivid and significant.  
It all goes well, but many of those words are for me just one occurrences: so there was just one place where I have
found tose words, put into Anki, and now struggling to remember them.

So my basic idea is to use the data from https://en.wiktionary.org/wiki/Wiktionary:Frequency_lists/Spanish . 
This way I can filter out those seldom words, and put more attention to the words that are used frequently. 

# What does it have to do with the programming? 

The data obtained from https://en.wiktionary.org/wiki/Wiktionary:Frequency_lists/Spanish were in a strange data format:
`word1_count word1 word2_count word2 word3_count word3 word4_count word4`. What I needed to enrich a script, was a
different format of the file:
```
word1_count word1 
word2_count word2 
word3_count word3 
word4_count word4
```
To avoid doing this manually, I have been trying to use ItellJ `Replace in Files` that supports multiline replacements. 
So I in a find input I have typed: 
```
(?<t>\d\d*?)
```




