
<!--
includes/sql-database-create-new-database-portal.md

Latest Freshness check:  2016-04-11 , carlrab.

As of circa 2016-04-11, the following topics might include this include:
articles/sql-database/sql-database-get-started-tutorial.md

-->
## <a name="create-a-new-azure-sql-database"></a>새 Azure SQL 데이터베이스 만들기
Azure 포털에서 다음 단계를 사용하여 새로운 또는 기존 Azure SQL 데이터베이스 논리 서버에 새 Azure SQL 데이터베이스를 만듭니다.

1. 연결되어 있지 않은 경우 [Azure Portal](http://portal.azure.com)에 연결합니다.
2. **새로 만들기**를 클릭하고 **SQL Database**를 입력한 다음 **SQL Database(새 데이터베이스)**를 클릭합니다.
   
     ![새 데이터베이스](./media/sql-database-create-new-database-portal/sql-database-create-new-database-portal-1.png)
3. **SQL Database(새 데이터베이스)**를 클릭합니다.
   
     ![새 데이터베이스](./media/sql-database-create-new-database-portal/sql-database-create-new-database-portal-2.png)
4. **만들기** 를 클릭하여 SQL Database 서비스에 새 데이터베이스를 만듭니다.
   
     ![새 데이터베이스](./media/sql-database-create-new-database-portal/sql-database-create-new-database-portal-3.png)
5. 다음 서버 속성에 대한 값을 제공합니다.
   
   * 데이터베이스 이름
   * 구독: 여러 개의 구독이 있는 경우에만 적용합니다.
   * 리소스 그룹: 시작하려면 논리 서버의 리소스 그룹을 사용합니다.
   * 원본 선택: 빈 데이터베이스, 샘플 데이터 또는 Azure 데이터베이스 백업을 선택할 수 있습니다. 온-프레미스 SQL Server 데이터베이스를 마이그레이션하거나 BCP 명령줄 도구를 사용하여 데이터를 로드하려면 이 문서의 끝에 있는 링크를 참조하세요.
   * 서버: 신규 또는 기존 논리 서버입니다.
   * 서버 관리자 로그인
   * 암호
   * 가격 책정 계층: 시작하려면 기본값 S0을 사용합니다.
   * 데이터 정렬: 빈 데이터베이스를 선택한 경우에만 적용합니다.
     
        ![새 데이터베이스](./media/sql-database-create-new-database-portal/sql-database-create-new-database-portal-4.png)
6. **만들기**를 클릭합니다. 알림 영역에서 배포가 시작된 것을 확인할 수 있습니다.
   
    ![새 데이터베이스](./media/sql-database-create-new-database-portal/sql-database-create-new-database-portal-5.png)
7. 다음 단계를 계속하기 전에 배포가 완료되기를 기다립니다.
   
     ![새 데이터베이스](./media/sql-database-create-new-database-portal/sql-database-create-new-database-portal-6.png)



<!--HONumber=Jan17_HO3-->


