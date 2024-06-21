---
title: "基于Legado书源规则生成python代码"
date: 2024-02-16T15:26:01+08:00
tags:
  - python
  - 基础
categories:
  - python
---

# 基于Legado书源规则生成python代码

> 书源规则：https://celetor.github.io/teachme/Rule/source.html
> 
> DrissionPage查找规则 https://g1879.gitee.io/drissionpagedocs/get_elements/usage


## python代码基础抽象类

```python
# -*- coding: utf-8 -*-
"""
@software: PyCharm
@file: base.py
@time: 2024/2/16 9:55
@author SuperLazyDog
"""
from abc import ABC, abstractmethod
from pydantic import BaseModel, Field
from typing import List, Optional
from DrissionPage import SessionPage


class SearchResult(BaseModel):
    name: str = Field(..., title="书名")
    bookUrl: str = Field(..., title="书籍链接")
    author: Optional[str] = Field(None, title="作者")
    coverUrl: Optional[str] = Field(None, title="封面链接")


class BookInfo(BaseModel):
    url: str = Field(..., title="书籍链接")
    name: str = Field(..., title="书名")
    author: str = Field(..., title="作者")
    intro: str = Field(..., title="简介")
    coverUrl: Optional[str] = Field(None, title="封面链接")
    kind: Optional[str] = Field(None, title="分类")
    lastChapter: Optional[str] = Field(None, title="最新章节")
    wordCount: Optional[str] = Field(None, title="字数")


class ChapterInfo(BaseModel):
    chapterName: str = Field(..., title="章节名")
    chapterUrl: str = Field(..., title="章节链接")


class Base(ABC):
    page = SessionPage(timeout=3)

    @property
    @abstractmethod
    def bookSourceName(self) -> str:
        """
        书源名称
        :return:
        """
        pass

    @property
    @abstractmethod
    def bookSourceUrl(self) -> str:
        """
        书源链接
        :return:
        """
        pass

    @abstractmethod
    def searchBook(self, keyword: str) -> List[SearchResult]:
        """
        搜索书籍
        :param keyword:
        :return:
        """
        pass

    @abstractmethod
    def getBookInfo(self, url: str) -> BookInfo:
        """
        获取书籍信息
        :param url:  书籍链接
        :return:
        """
        pass

    @abstractmethod
    def getChapterInfo(self, url: str, result=None) -> List[ChapterInfo]:
        """
        获取章节信息
        :param url:  章节页面链接
        :param result:  结果
        :return:
        """
        pass

    @abstractmethod
    def getContent(self, info: ChapterInfo, content: str = None) -> str:
        """
        获取章节内容
        :param info: 章节信息
        :param content: 章节内容
        :return:
        """
        pass

```

## 69书吧示例
### 书源规则
```json
{
    "bookSourceComment": "/*\nBy_zhbyjm7783\neval(String(source.bookSourceComment))\n*/\n\t\nvar error=/429 Too Many Requests/;\nwhile(error.test(result)){\n\tcookie.removeCookie(baseUrl);\n\tresult=java.ajax(baseUrl);\n\t}\n\t\tresult;",
    "bookSourceGroup": "源仓库",
    "bookSourceName": "69书吧[自写]",
    "bookSourceType": 0,
    "bookSourceUrl": "https://69shu.net",
    "bookUrlPattern": "https://69shu.net/\\d+.html",
    "customOrder": -240,
    "enabled": true,
    "enabledCookieJar": false,
    "enabledExplore": true,
    "exploreUrl": "首页::/\n排行::/top/lastupdate/{{page}}/\n周点击榜::/top/weekvisit/{{page}}/\n月点击榜::/top/monthvisit/{{page}}/\n总点击榜::/top/allvisit/{{page}}/\n推荐榜::/top/allvote/{{page}}/\n必看榜::/hot/\n完本::/quanben/{{page}}/",
    "header": "",
    "lastUpdateTime": 1701848258494,
    "loginUrl": "",
    "respondTime": 109239,
    "ruleBookInfo": {
        "author": "[property=\"og:novel:author\"]@content",
        "coverUrl": "[property=\"og:image\"]@content",
        "init": "",
        "intro": "[property=\"og:description\"]@content",
        "kind": "[property~=status|update_time]@content",
        "lastChapter": "[property=\"og:novel:latest_chapter_name\"]@content",
        "name": "[property=\"og:novel:book_name\"]@content",
        "tocUrl": "",
        "wordCount": ".word-count@text"
    },
    "ruleContent": {
        "content": "<js>\neval(String(source.bookSourceComment))\n</js>.novelcontent@html"
    },
    "ruleExplore": {
        "author": ".author@text",
        "bookList": ".xbk",
        "bookUrl": "a.0@href",
        "coverUrl": "img@data-src",
        "intro": ".intro@text",
        "name": ".listtext@h2@a@text"
    },
    "ruleSearch": {
        "author": "em@text",
        "bookList": ".search_list",
        "bookUrl": "a@href",
        "checkKeyWord": "",
        "coverUrl": "a@href<js>\nvar id = result.match(/(\\d+)\\/?$/)[1];\nvar idi=(id)\nvar iid = parseInt(idi/1000);\n'https://img.69shu.net/'+iid+'/'+idi+'/'+idi+'s.jpg';\n</js>",
        "intro": "",
        "kind": "",
        "lastChapter": "",
        "name": "a@text"
    },
    "ruleToc": {
        "chapterList": "<js>\neval(String(source.bookSourceComment))\n</js>.p2.1@li",
        "chapterName": "a@text",
        "chapterUrl": "a@href",
        "nextTocUrl": "option@value||text.下一页@href"
    },
    "searchUrl": "https://69shu.net/s.php,{\n  \"charset\": \"gbk\",\n  \"method\": \"POST\",\n  \"body\": \"s={{key}}&type=articlename\"\n}",
    "weight": 0
}
```
### python代码
```python
import re
from utils import logger
from utils.decorator import retry
from ._base_ import Base, SearchResult, List, BookInfo, ChapterInfo


class Shu69(Base):
    bookSourceUrl = "https://69shu.net"
    bookSourceName = "69书吧[自写]"

    @retry()
    def searchBook(self, keyword: str) -> List[SearchResult]:
        searchUrl = "https://69shu.net/s.php"
        data = {"s": keyword.encode("gbk"), "type": "articlename"}
        self.page.post(searchUrl, data=data)
        bookList = self.page.eles(".search_list")
        result = list()
        for book in bookList:
            coverUrl = None
            if book.ele("tag:z") and book.ele("tag:z").attr("href") is not None:
                coverUrl = book.ele("tag:z").attr("href")
                _id = re.search(r"(\d+)/?$", coverUrl).group(1)
                idi = int(_id)
                iid = int(idi / 1000)
                coverUrl = f"https://img.69shu.net/{iid}/{idi}/{idi}s.jpg"
            result.append(
                SearchResult(
                    name=book.ele("tag:a").text,
                    author=book.ele("tag:em").text if book.ele("tag:em") else None,
                    coverUrl=coverUrl,
                    bookUrl=book.ele("tag:a").attr("href"),
                )
            )
        return result

    @retry()
    def getBookInfo(self, url: str) -> BookInfo:
        self.page.get(url)
        return BookInfo(
            url=url,
            name=self.page.ele("@property=og:novel:book_name").attr("content"),
            author=self.page.ele("@property=og:novel:author").attr("content"),
            coverUrl=self.page.ele("@property=og:image").attr("content"),
            intro=self.page.ele("@property=og:description").attr("content"),
            kind=self.page.ele("@|property:status@|property:update_time").attr(
                "content"
            ),
            lastChapter=self.page.ele("@property=og:novel:latest_chapter_name").attr(
                "content"
            ),
            wordCount=self.page.ele(".word-count").text,
        )

    @retry()
    def getChapterInfo(self, url: str, result=None) -> List[ChapterInfo]:
        if result is None:
            result = list()
        self.page.get(url)
        chapterList = self.page.ele(".p2", index=2).eles("tag:li")
        nextTocUrl = (
            self.page.ele("option").attr("value")
            if self.page.ele("option")
            else (
                self.page.ele("text=下一页").attr("href")
                if self.page.ele("text=下一页")
                else None
            )
        )
        for chapter in chapterList:
            result.append(
                ChapterInfo(
                    chapterName=chapter.ele("tag:a").text,
                    chapterUrl=chapter.ele("tag:a").attr("href"),
                )
            )
        if nextTocUrl:
            logger.debug(f"nextTocUrl: {nextTocUrl}")
            self.getChapterInfo(nextTocUrl, result)
        return result

    @retry()
    def getContent(self, info: ChapterInfo, content: str = None) -> str:
        content = content if content else ""
        self.page.get(info.chapterUrl)
        content += "".join(
            [
                item.strip() + "\n"
                for item in self.page.ele(".novelcontent").text.split("\n") if item.strip()
            ]
        )
        return content

```

## 万相书城
### 书源规则
```json
{
    "bookSourceGroup": "源仓库",
    "bookSourceName": "万相书城",
    "bookSourceType": 0,
    "bookSourceUrl": "https://www.wxscn.com/",
    "customOrder": 0,
    "enabled": true,
    "enabledCookieJar": false,
    "enabledExplore": false,
    "lastUpdateTime": 1698759595973,
    "respondTime": 34062,
    "ruleBookInfo": {
        "author": "class.col-md-8 col-sm-6 dark.0@tag.a.0@text",
        "intro": "class.col-sm-11 col-xs-10@text",
        "lastChapter": "class.col-md-8 col-sm-6 dark.1@tag.a.0@text",
        "name": "class.book-name@text",
        "tocUrl": "class.panel-footer visible-xs visible-sm@href"
    },
    "ruleContent": {
        "content": "id.cont-body@text",
        "nextContentUrl": "class.col-md-6 text-center@tag.a.2@href"
    },
    "ruleSearch": {
        "author": "",
        "bookList": "class.table table-condensed@tag.tr!0",
        "bookUrl": "a.0@href",
        "checkKeyWord": "",
        "name": "a.0@text"
    },
    "ruleToc": {
        "chapterList": "class.col-md-6 item",
        "chapterName": "class.col-md-6 item@tag.a@text",
        "chapterUrl": "class.col-md-6 item@tag.a@href"
    },
    "searchUrl": "https://www.wxscn.com/plus/search.php?q={{key}}",
    "weight": 0
}
```
### python代码
```python

from utils.decorator import retry
from ._base_ import Base, SearchResult, List, BookInfo, ChapterInfo


class Wxscn(Base):
    bookSourceName = "万相书城"
    bookSourceUrl = "https://www.wxscn.com/"

    @retry()
    def searchBook(self, keyword: str) -> List[SearchResult]:
        searchUrl = "https://www.wxscn.com/plus/search.php"
        params = {"q": keyword}
        self.page.get(searchUrl, params=params)
        bookList = self.page.ele("@class:table table-condensed").eles("tag=tr")[1:]
        print(len(bookList))
        result = list()
        for book in bookList:
            result.append(
                SearchResult(
                    name=book.ele("tag=a").text,
                    bookUrl=book.ele("tag=a").attr("href"),
                )
            )
        return result

    @retry()
    def getBookInfo(self, url: str) -> BookInfo:
        self.page.get(url)
        return BookInfo(
            url=url,
            name=self.page.ele("@class:book-name").text.strip(),
            author=self.page.eles("@class:col-md-8 col-sm-6 dark")[0]
            .eles("tag=a")[0]
            .text.strip(),
            intro=self.page.ele("@class:col-sm-11 col-xs-10").text.strip(),
            lastChapter=self.page.eles("@class:col-md-8 col-sm-6 dark")[1]
            .eles("tag=a")[0]
            .text,
        )

    @retry()
    def getChapterInfo(self, url: str, result=None) -> List[ChapterInfo]:
        if result is None:
            result = list()
        self.page.get(url)
        chapterList = self.page.eles("@class:col-md-6 item")
        for chapter in chapterList:
            result.append(
                ChapterInfo(
                    chapterName=chapter.ele("tag=a").text,
                    chapterUrl=chapter.ele("tag=a").attr("href"),
                )
            )
        return result

    @retry()
    def getContent(self, info: ChapterInfo, content: str = None) -> str:
        content = content if content else ""
        self.page.get(info.chapterUrl)
        content += "".join(
            [
                item.strip() + "\n"
                for item in self.page.ele("@id=cont-body").text.split("\n") if item.strip()
            ]
        )
        return content

```

## 笔趣岛
### 书源规则
```json
{
    "bookSourceGroup": "源仓库",
    "bookSourceName": "笔趣岛",
    "bookSourceType": 0,
    "bookSourceUrl": "https://m.biqudao.cc",
    "bookUrlPattern": "https?://m.biqudao.cc/book/[\\d_]+/",
    "customOrder": 0,
    "enabled": true,
    "enabledCookieJar": true,
    "enabledExplore": true,
    "exploreUrl": "全部小说::/xclass/0/{{page}}.html\n玄幻小说::/xclass/1/{{page}}.html\n修真小说::/xclass/2/{{page}}.html\n都市小说::/xclass/3/{{page}}.html\n穿越小说::/xclass/4/{{page}}.html\n网游小说::/xclass/5/{{page}}.html\n科幻小说::/xclass/6/{{page}}.html\n其他小说::/xclass/7/{{page}}.html\n全本小说::/quanben_{{page}}.html",
    "header": "{\"User-Agent\": \"Mozilla/5.0 (Linux; Android 9) Mobile Safari/537.36\"}",
    "lastUpdateTime": 1701773629170,
    "respondTime": 45348,
    "ruleBookInfo": {
        "author": "p.author@text",
        "coverUrl": ".synopsisArea_detail img@src",
        "intro": "p.review@text##简介：",
        "kind": ".synopsisArea_detail p.1:2:3@text##.*：",
        "lastChapter": ".synopsisArea_detail a.-1@text",
        "name": "span.title@text"
    },
    "ruleContent": {
        "content": "#chaptercontent@textNodes",
        "nextContentUrl": "text.下一页@href",
        "replaceRegex": "##\\s*{{try{chapter.title}catch(e){\"\"} }}.*\\s*"
    },
    "ruleExplore": {
        "author": "p.author@text",
        "bookList": ".hot_sale",
        "bookUrl": "a@href",
        "coverUrl": "img@data-original",
        "intro": "p.review@text##简介：",
        "kind": "0",
        "name": "p.title@text"
    },
    "ruleSearch": {
        "author": "p.author.0@text##.*\\|",
        "bookList": ".hot_sale",
        "bookUrl": "a@href",
        "checkKeyWord": "剑来",
        "coverUrl": "@js:\"https://m.biqudao.cc/files/article/image/24/24458/24458s.jpg\"",
        "kind": "p.author@text##\\|.*",
        "lastChapter": "p.author.1@text##.*\\|\\s*更新：",
        "name": "p.title@text"
    },
    "ruleToc": {
        "chapterList": ".directoryArea.1@p a",
        "chapterName": "text",
        "chapterUrl": "href",
        "nextTocUrl": "option@value||text.下一页@href"
    },
    "searchUrl": "/s.php,{\n  \"body\": \"keyword={{key}}&t=1\",\n  \"method\": \"POST\"\n}",
    "weight": 0
}

```
### python代码
```python

from utils import logger
from utils.decorator import retry
import re
from ._base_ import Base, SearchResult, List, BookInfo, ChapterInfo


class BiQuDao(Base):
    bookSourceName = "笔趣岛"

    bookSourceUrl = "https://m.biqudao.cc"

    @retry()
    def searchBook(self, keyword: str) -> List[SearchResult]:
        searchUrl = "https://m.biqudao.cc/s.php"
        data = {"keyword": keyword, "t": 1}
        self.page.post(searchUrl, data=data)
        bookList = self.page.eles("@class:hot_sale")
        result = list()
        for book in bookList:
            result.append(
                SearchResult(
                    name=book.ele("tag=p@class:title").text,
                    bookUrl=book.ele("tag=a").attr("href"),
                    author=book.ele("tag=p@class:author").text.split("|")[1],
                )
            )
        return result

    @retry()
    def getBookInfo(self, url: str) -> BookInfo:
        self.page.get(url)
        return BookInfo(
            url=url,
            name=self.page.ele("tag=span@class:title").text,
            author=self.page.ele("tag=p@class:author").text.split("|")[0],
            intro=self.page.ele("tag=p@class:review").text.replace("简介：", ""),
            coverUrl=self.page.ele("@class:synopsisArea_detail")
            .ele("tag=img")
            .attr("src"),
            kind=self.page.ele("@class:synopsisArea_detail")
            .ele("tag=p", index=3)
            .text.split("：")[1],
            lastChapter=self.page.ele("@class:synopsisArea_detail")
            .ele("tag=p", index=-1)
            .text,
        )

    @retry()
    def getChapterInfo(self, url: str, result=None) -> List[ChapterInfo]:
        if result is None:
            result = list()
        self.page.get(url)
        nextTocUrl = (
            self.page.ele("option").attr("value")
            if self.page.ele("option")
            else (
                self.page.ele("text=下一页").attr("href")
                if self.page.ele("text=下一页")
                else None
            )
        )
        chapterList = self.page.ele("@class:directoryArea", index=2).eles("tag=a")
        for chapter in chapterList:
            result.append(
                ChapterInfo(
                    chapterName=chapter.text,
                    chapterUrl=chapter.attr("href"),
                )
            )
        if nextTocUrl:
            logger.debug(f"nextTocUrl: {nextTocUrl}")
            self.getChapterInfo(nextTocUrl, result)
        return result

    @retry()
    def getContent(self, info: ChapterInfo, content: str = None) -> str:
        content = content if content else ""
        self.page.get(info.chapterUrl)
        nextContentUrl = (
            self.page.ele("text=下一页").attr("href")
            if self.page.ele("text=下一页")
            else None
        )
        reg = re.compile(rf"\s*{info.chapterName}.*\s*")

        content += reg.sub("", "".join(
            [
                item.strip() + "\n"
                for item in self.page.ele("@id=chaptercontent").text.split("\n") if item.strip()
            ]
        ))
        if nextContentUrl:
            logger.debug(f"nextContentUrl: {nextContentUrl}")
            info.chapterUrl = nextContentUrl
            return self.getContent(info, content)
        return content

```
