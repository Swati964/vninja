---
type: "post"
draft: "true"
date: "{{ dateFormat "2006-01-02" .Date }}"
year: "{{ dateFormat "2006" .Date }}"
author: Christian Mohn
keywords: ["key", "words"]
topics: ["topic 1"]
tags: ["one", "two"]

title: "{{ replace .TranslationBaseName "-" " " | title }}"

description: "description"

FeaturedImage: "https://vninja.net/"

twitter:
card: "summary_large_image"
site: "@vninjanet"
creator: "@h0bbel" 
---
