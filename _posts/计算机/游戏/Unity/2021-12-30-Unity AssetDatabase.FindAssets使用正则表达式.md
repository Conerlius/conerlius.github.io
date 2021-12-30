---
layout:     article
title:      "unity AssetDatabase.FindAssets使用正则表达式"
subtitle:   " \"unity\""
date:       2021-12-30 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
    - AssetDatabase
---

`google`上找了一圈，没有找到这样的问题，看到的回答对我都没有啥帮助，就自己写了一个！

这是个小问题，就不废话了！

先看看`AssetDatabase.FindAssets`的api

```c#
public static string[] FindAssets(string filter, string[] searchInFolders)
{
  SearchFilter searchFilter = new SearchFilter()
  {
    searchArea = SearchFilter.SearchArea.AllAssets
  };
  SearchUtility.ParseSearchString(filter, searchFilter);
  if (searchInFolders != null && (uint) searchInFolders.Length > 0U)
  {
    searchFilter.folders = searchInFolders;
    searchFilter.searchArea = SearchFilter.SearchArea.SelectedFolders;
  }
  return AssetDatabase.FindAssets(searchFilter);
}
```

这里能看到`FindAssets`实际上是通过`SearchUtility.ParseSearchString`去生成`filter`的。

```c#
internal static bool ParseSearchString(string searchText, SearchFilter filter)
{
  if (string.IsNullOrEmpty(searchText))
    return false;
  filter.ClearSearch();
  filter.originalText = searchText;
  string searchString1 = string.Copy(searchText);
  // 这里是将": "替换成":"
  SearchUtility.RemoveUnwantedWhitespaces(ref searchString1);
  bool flag = false;
  int startIndex1 = SearchUtility.FindFirstPositionNotOf(searchString1, " \t,*?");
  if (startIndex1 == -1)
    startIndex1 = 0;
  int num1;
  for (; startIndex1 < searchString1.Length; startIndex1 = num1 + 1)
  {
    num1 = searchString1.IndexOfAny(" \t,*?".ToCharArray(), startIndex1);
    if (num1 == -1)
      num1 = searchString1.Length;
    int num2 = searchString1.IndexOf('"', startIndex1);
    int startIndex2 = -1;
    if (num2 != -1 && num2 < num1)
    {
      startIndex2 = searchString1.IndexOf('"', num2 + 1);
      if (startIndex2 != -1 && startIndex2 > num1 - 1)
      {
        num1 = searchString1.IndexOfAny(" \t,*?".ToCharArray(), startIndex2);
        if (num1 == -1)
          num1 = searchString1.Length;
      }
    }
    if (num1 > startIndex1)
    {
      string searchString2 = searchString1.Substring(startIndex1, num1 - startIndex1);
      if (SearchUtility.CheckForKeyWords(searchString2, filter, num2 - startIndex1, startIndex2 - startIndex1))
      {
        flag = true;
      }
      else
      {
        SearchFilter searchFilter = filter;
        searchFilter.nameFilter = searchFilter.nameFilter + (string.IsNullOrEmpty(filter.nameFilter) ? "" : " ") + searchString2;
      }
    }
  }
  return flag;
}
```

大致代码如上，官方文档说明是这样的 https://docs.unity3d.com/2021.1/Documentation/Manual/search-filters.html

所以如果想要在`AssetDatabase.FindAssets`中使用正则，就需要按照官方文档这样子说明使用才可以。