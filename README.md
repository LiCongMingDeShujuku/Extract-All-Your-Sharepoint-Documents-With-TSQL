![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 使用TSQL提取所有Sharepoint文件
#### Extract All Your Sharepoint Documents With TSQL

![#](images/##############?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
这是我之前创建的一些SQL逻辑，它将在你的所有Sharepoint内容数据库中构建一个所有文档（doc，docx，pdf，txt，msg）的列表。在这个阶段它可能会使用一些updating。我使用了没有程序或功能的纯SQL，尽管我确实知道程序和功能
的好处;我只是没有记住这些功能。如果你需要改善或者改进，请务必留评论说明。它远非完美，ye不优雅，但确实有效。此过程非常适用于手动文档提取和归档，合规性调整，容量管理等。此过程并非完全自动化。它构建了一组提取脚
本（xml格式），因此你只需单击xml链接即可获得所有脚本。你可以一次性把他们运行完，储存在一个表中，或者只取某些你需要的提取脚本，都是可以的。你只需要修改底部的WHERE从句。我还包含了几个@wildcard变量，你可以根据需要查询更精细的结果。

那么它是如何工作的？
它使用数据库名称，列表名称，站点URL，文档名称和ID构建列表。它会过滤doc，docx，pdf，txt和msg，但如果需要，你可以轻松添加或删除不同的文档类型。它获取此信息并将其全部拉入名为## sample_collection的临时表中。
此后的所有进程都是基于临时表的，因此你的Sharepoint数据库在提取开始之前基本上是不受影响的，它只是一个读取操作来获取文档二进制文件，然后最终下载文档。如果你有数百万个2mb文件（就像我一样），那么下载开始时会非常快。 如果你有一个大的Sharepoint环境，在运行此程序之前，请确保你的Temp空间达到最佳状态。 大部分的负担都在Temp上，并且在获得所有脚本时只需要很少时间，所以请耐心等待。

顺便说一下; 此示例中的文件夹结构将转以下路径：
\\ MyServerName\ W$\ BACKUPS\ Sharepoint_Extraction
所有其他文件夹都会被添加到其中。

如果需要，你随时可以无风险地终止程序。而且， 它不会自动运行。它会为你呈现提取脚本。它会创建与文档所在的站点URL相同的文件夹结构。文件夹结构非常容易在逻辑（logic）中识别。每个文件夹都会根据数据库服务器、实例名称和Sharepoint表，以其用途为前缀。
例如; 它始终以实例名称（instance name）开头，并以文档名称（document name）结尾。

SRV_MyServerName_MyInstanceName\DB_MyDatabaseName\LIST_MyListName\URL_MyURLInFolderFormat\MyDocument.pdf

如果你的URL中有长路径，那么文档的文件夹结构会同样深。文件夹结构会删除逗号和连字符，但你需要添加更多REPLACE语句来容纳任何其他的特殊字符。我还在所有变量上使用了6个字符的后缀，这些变量是使用[id]的（前3个和后3个）字符创建的，以防止变量冲突。你会看到很多的重复运行。这其实可以通过函数或程序更好地处理，但如果可能的话我不喜欢创建任何其他附加对象，特别是对于管理或维护过程。保持它纯SQL的形式，在将其应用于其他环境时会非常便携。为了保持轻量级，我继续在脚本上添加了一（select top10），这样你就可以在运行完整个set之前看到输出的样子。就像我说的，它的确需要一些时间来制作脚本。你只需删除“top 10”就可以了。
这就是对此的总结。希望你觉得它有用。

这是逻辑（logic）：

## English
Ok so here’s some SQL logic I created a while back that will build a list of all your docs (doc, docx, pdf, txt, msg) across all your Sharepoint Content Databases. It could probably use a bit of updating at this stage. This is using straight SQL with no procedures or functions although I do understand the benefits there; I just didn’t write this with those in mind. If you have modifications, or improvements; please by all means –drop in a comment. It’s far from perfect, and definitely not elegant, but it does work. This process is excellent for manual document extraction and archiving, compliance sleuthing, capacity management, etc. This process is not entirely automatic. It builds a set of extraction scripts (in xml form) so all you need to do is click on the xml link and you’ll get all scripts. It’s up to you if you want to run them all at once, store them in a table, or get only certain extraction scripts for what you’re looking for. All you need to do is modify the WHERE clause at the bottom. I’ve also included a couple @wildcard variables for more granular results on your queries.

How does it work?
It builds a list using the Database Name, List Name, Site URL, Document Name, and ID. It filters for doc, docx, pdf, txt, and msg but you can easily add or remove different document types if needed.
It takes this information and pulls it all into a Temp table called ##sample_collection. All processes thereafter are dependent on the temp table so your Sharepoint database are largely untouched until extraction begins and even then; it’s just a read operation to get the document binaries, and then the document is ultimately downloaded. If you have millions of 2mb files (like I do), it’s surprisingly quick when the downloads begin. If you have a large Sharepoint environment; make sure your Temp space is up to scratch before running this process. Most of the burden is on the Temp and it takes alittle while to get all the scripts produced so just be patient for that.

By the way; the folder structure in this example is going to this path:
\\MyServerName\W$\BACKUPS\Sharepoint_Extraction
All the other folders are added to it.

You can always stop the process with no risk if needed. Again; this does NOT automatically run. It presents the extraction scripts to you.
It creates an identical folder structure to that of the Site URL where the documents exist. The folder structure is readily identified within the logic. Each folder is prefixed by it’s purpose according to Database Server, Instance Name, and Sharepoint tables.
For example; it always begins with the instance name, and end with the document name.

SRV_MyServerName_MyInstanceName\DB_MyDatabaseName\LIST_MyListName\URL_MyURLInFolderFormat\MyDocument.pdf

If you have long paths in the URL; expect an equally deep folder structure for the documents. The folder structure removes commas and hyphens, but you’ll need to add more REPLACE statements to accommodate any other special characters.
I also used a 6 character suffix on all variables which is created by using the (first 3 + last 3) characters of the [id] to prevent any variable conflict. You’ll see a lot of that repeated. This could have been handled better with a function or procedure but I don’t like creating any other additional objects if at all possible especially for administrative, or maintenance processes. Keeping it straight SQL keeps it extremely portable when applying it to other environments.
To keep things lightweight; I went ahead and added a (select top 10) on the script so you can see what the output looks like before you run the full set. Like I said it does take some time produce scripts. Just remove the ‘top 10’ and you should be all set.
Anyway; so that’s the run-down on this. Hope you find it helpful.

Here’s the logic:

---
## Logic
```SQL
use master;
set nocount on
 
--		check for advanced options and ole automation
--		检查高级选项和自动化
declare @adv_options    int
declare @ole_automation int
set     @adv_options    = (select cast([value_in_use] as int) from sys.configurations where [configuration_id] = '518')
set     @ole_automation = (select cast([value_in_use] as int) from sys.configurations where [configuration_id] = '16388') 
if      @adv_options    = 0 begin exec master..sp_configure 'show advanced options', 1;     reconfigure with override; end
if      @ole_automation = 0 begin exec master..sp_configure 'Ole Automation Procedures', 1; reconfigure with override; end
 
--		build logic to get file info (doc, docx, pdf, txt) across all content databases
--		构建逻辑（logic），在所有内容数据库中获取文件信息（doc，docx，pdf，txt）
declare @wildcard1      varchar(255)
declare @wildcard2      varchar(255)
declare @get_file_info  varchar(max)
set     @wildcard1      = '%design%'
set     @wildcard2      = '%design%'
set     @get_file_info  = ''
select  @get_file_info  = @get_file_info +
'
use [' + [name] + '];' + char(10) + 
'
 select 
    ''database''            = db_name()
,   ''time_created''        = alldocs.timecreated
,   ''list_name''           = alllists.tp_title
,   ''file_name''           = alldocs.leafname
,   ''url''                 = alldocs.dirname
,   ''id''                  = cast(alldocs.id as varchar(255))
from 
    alldocs join alldocstreams  on alldocs.id=alldocstreams.id 
    join alllists               on alllists.tp_id = alldocs.listid
where
    alldocs.leafname like       ''' + @wildcard1 + '''
    or  alldocs.dirname like    ''' + @wildcard2 + '''
    and right(alldocs.leafname, 2) in (''oc'', ''cx'', ''df'', ''xt'', ''sg'')
'
from    sys.databases where [name] like '%content%'
 
--		build temp table for results (this will take several minutes)
--		创建临时表格记录结果（需要花费几分钟）
if object_id('tempdb..##sample_collection') is null
    begin
        create table    ##sample_collection ([database] varchar(255), [time_created] datetime, [list_name] varchar(255), [file_name] varchar(510), [url] varchar(510), [id] varchar(255))
        insert into     ##sample_collection exec    (@get_file_info)
    end;
 
--		get count of documents in table
--		获取表格中的文件数量
select count(*) from ##sample_collection
 
--		build extraction process including folder structure. this will take several minutes.
--		构建提取程序，包含文件夹结构，这将需要花费几分钟。
declare @file_extraction    varchar(max)
set     @file_extraction    = ''
select  top 10 @file_extraction = @file_extraction +
'
use [' + [database] + '];
set nocount on;' + char(10) + '
 
declare     @folder'    + '_' + left(id, 3) + right(id, 3) + ' nvarchar(4000) 
set         @folder'    + '_' + left(id, 3) + right(id, 3) + ' = N''\\MyServerName\W$\BACKUPS\Sharepoint_Extraction\' 
            + 'SVR_'    + replace(upper(@@servername), '\', '_') + '\' 
            + 'DB_'     + upper([database])     + '\' 
            + 'LIST_'   + upper([list_name])    + '\' 
            + 'URL_'    + replace(replace(replace(upper([url]), '/', '\'), '''', ''), ',', '') + '''
exec        master..xp_create_subdir @folder' + '_' + left(id, 3) + right(id, 3) + '; 
 
declare @object_token'      + '_' + left(id, 3) + right(id, 3) + ' int
declare @destination_path'  + '_' + left(id, 3) + right(id, 3) + ' varchar(255)
declare @content_binary'    + '_' + left(id, 3) + right(id, 3) + ' varbinary(max)
set     @destination_path'  + '_' + left(id, 3) + right(id, 3) + ' = (select @folder' + '_' + left(id, 3) + right(id, 3) + ' + ''\' +  [file_name] + ''')
select  @content_binary'    + '_' + left(id, 3) + right(id, 3) + ' = alldocstreams.content from alldocs join alldocstreams on alldocs.id = alldocstreams.id join alllists on alllists.tp_id = alldocs.listid
where  
    alldocs.leafname    = ''' + [file_name] + '''
    and alldocs.dirname = ''' + [url]       + '''
 
exec sp_oacreate ''adodb.stream'', @object_token' + '_' + left(id, 3) + right(id, 3) + ' output
exec sp_oasetproperty   @object_token' + '_' + left(id, 3) + right(id, 3) + ', ''type'', 1
exec sp_oamethod        @object_token' + '_' + left(id, 3) + right(id, 3) + ', ''open''
exec sp_oamethod        @object_token' + '_' + left(id, 3) + right(id, 3) + ', ''write'',       null, @content_binary'      + '_' + left(id, 3) + right(id, 3) + '
exec sp_oamethod        @object_token' + '_' + left(id, 3) + right(id, 3) + ', ''savetofile'',  null, @destination_path'    + '_' + left(id, 3) + right(id, 3) + ', 2
exec sp_oamethod        @object_token' + '_' + left(id, 3) + right(id, 3) + ', ''close''
exec sp_oadestroy       @object_token' + '_' + left(id, 3) + right(id, 3) + '
'
from ##sample_collection where right([file_name], 2) in ('oc', 'cx', 'df', 'xt', 'sg')
 
select  (@file_extraction) for xml path(''), type
 
--	drop table ##sample_collection
--	删除表格##sample_collection
```
One point I would like to make is I noticed that some documents (both pdf and doc) will periodically be (or appear to be) empty files, but the size of the file usually shows bytes so there should be some data in there. For whatever reason; I’m not able to see the content post download. This has been random, and it could be that teh files are checked out, but I cannot say for certain. It’s been maybe 1 of every 200 documents or so. If anyone has tips on what is causing this; please post a comment. 
我注意到一些文档（pdf和doc）将定期（或看起来）是空文件，但文件的大小通常显示有字节，因此应该有一些数据。不管出于什么原因我无法看到下载后的内容。这是随机的，可能是文件被检出，但我不确定。这可能是每200份文件中的一份左右。如果有人知道造成这种情况的原因，请发表评论。

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

