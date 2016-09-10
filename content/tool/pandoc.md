<!--
@Author: xiewenqian <int>
@Date:   2016-09-10T14:05:41+08:00
@Email:  wixb50@gmail.com
@Last modified by:   int
@Last modified time: 2016-09-10T14:25:17+08:00
-->


---
title: "pandoc"
date: 2016-09-10 14:06
collection: "文本处理"
---

## 介绍

强大的文本转化工具.

## 支持的格式

```
Input formats:  commonmark, docbook, docx, epub, haddock, html, json*, latex,
                markdown, markdown_github, markdown_mmd, markdown_phpextra,
                markdown_strict, mediawiki, native, odt, opml, org, rst, t2t,
                textile, twiki
                [ *only Pandoc's JSON version of native AST]
Output formats: asciidoc, beamer, commonmark, context, docbook, docbook5, docx,
                dokuwiki, dzslides, epub, epub3, fb2, haddock, html, html5,
                icml, json*, latex, man, markdown, markdown_github,
                markdown_mmd, markdown_phpextra, markdown_strict, mediawiki,
                native, odt, opendocument, opml, org, pdf**, plain, revealjs,
                rst, rtf, s5, slideous, slidy, tei, texinfo, textile, zimwiki
                [**for pdf output, use latex or beamer and -o FILENAME.pdf]
```

## 使用方法

markdown转wiki
```
pandoc -f markdown -t mediawiki -o data.api data-apis.md
```
