title: ibatis批量插入
date: 2015-04-15 16:00:53
tags:
---

今天用到了ibatis批量插入, 代码如下

```{bash}
public void insertPgwUser(final List<xxx> dtoList) {
		 
		this.getSqlMapClientTemplate().execute(new SqlMapClientCallback() {
			public Object doInSqlMapClient(SqlMapExecutor executor) throws SQLException {
				executor.startBatch();
			 
				final int batchSize = 100000;
				int count = 0;
				for (xxx record : dtoList) {
					executor.insert("insert.in.addpassGuidwireUser", record);
					if (++count % batchSize == 0) {
						executor.executeBatch();
					}
				}
				 
				executor.executeBatch();
				return null;
			}
		});
	}
```

batchSize为一次插入条数