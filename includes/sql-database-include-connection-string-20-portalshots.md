
<!--
includes/sql-database-include-connection-string-20-portalshots.md

Latest Freshness check:  2015-09-02 , GeneMi.

## Connection string
-->


### <a name="obtain-the-connection-string-from-the-azure-portal"></a>從 Azure 入口網站取得連接字串
使用 [Azure 入口網站](https://portal.azure.com/)來取得用戶端程式與 Azure SQL Database 進行互動所需的連接字串。 

1. 選取 [所有服務] > [SQL 資料庫]。

2. 在 [SQL 資料庫] 刀鋒視窗左上角附近的篩選文字方塊中輸入您的資料庫名稱。

3. 選取資料庫的資料列。

4. 在刀鋒視窗顯示您的資料庫之後，為了閱讀方便，選取 [最小化] 按鈕來摺疊用於瀏覽和資料庫篩選的刀鋒視窗。 
   
5. 在您資料庫的刀鋒視窗上，選取 [顯示資料庫連接字串]。

6. 如果您想要使用 ADO.NET 連線庫，請複製標示為 **ADO**的字串。 
   
    ![複製資料庫的 ADO 連接字串][20-CopyAdoConnectionString]
7. 以其中一種格式，將連接字串資訊貼入您的用戶端程式碼中。

如需詳細資訊，請參閱[連接字串與設定檔](http://msdn.microsoft.com/library/ms254494.aspx)。

<!-- Image references. -->



[20-CopyAdoConnectionString]: ./media/sql-database-include-connection-string-20-portalshots/connqry-connstr-b.png


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->
