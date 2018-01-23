---
title: .net webform connection not close
date: 2018-01-23 14:27:27
tags:
---

>最近遇到最神奇的一件事，就算解決了還是覺得非常莫名  

# 大前提
>環境：.NET Framwork 5 、SQL Server 2012 、 ASP .NET webForm  
>問題：tempDB吃光，不管SSMS還是web都有問題

事情的開始是這樣的，因為有使用迴圈CTE(Common Table Expression)的需求，所以就用`Function` 把那一串 SQL 包起來，大概是這樣：

``` C
Function(){
    Connection open
    run CTE
    Connection close
}
```

呼叫的時候把結果再放進一個 function 當參數

``` C 
getFunc(Function()){
    Connection open
    string[] result = SQL result
    Connection close
    return result
}

```
最後把他直接放進DataSet裡面，丟給GridView

``` C
GridView.DataSource = GetDataSet(getFunc(Function()));
```

一開始測的時候都好好的，突然某一天發現SQL Server 掛掉了，連過去一看tempDB完全被吃光，好險在上線前就發現這個問題，不然把正式機用掛，重開機損失就大了。

中間各種查，文件各種看，確認CTE就像一般的SQL select 一樣，排序時會使用tempDB，另外用迴圈時會把**所有撈出來的資料**放在tempDB，最後使用 SQL Server Profiler 監控，終於發現是Conneciton沒有關閉的緣故( exec sp_reset_connection )。  

# 解決辦法

最後換了一個寫法，先用一個變數承接，再拿來使用
``` C 
var temp = getFunc(Function());
GridView.DataSource = GetDataSet(temp);

```
然後問題就解決的................

猜測大概是因為
<mark>（以下原因皆為腦補）還沒google到問題點</mark>  
recursive CTE 會使用tempDB  
connection close 的話會自動釋放  
直接傳入太多 function 會讓close 失效?(不確定)  

# 結論
下次先用變數接function return 的值

