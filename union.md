union all，解释为联合所有。

Union解释为联合。

union或者Union all实现把前后两个select集合的数据联合起来，组成一个结果集查询输出。这就要求联合前后的结果集，需要分别有相同的输出字段的数目，并且对应的字段类型要相同。
SELECT column1, column2 from table1 union (all) select column1, column2 from table2

以上语句要求量表的column1字段类型相同，column2类型相同。而且每个查询的数目都是一样的。UNION ALL和UNION的差别就在ALL上面，第一个叫联合所有，说明会显示前后两个查询所有的数据，而UNION没有ALL(所有)这个单词，实现将前后两个查询的数据联合到一起后，去掉重复的数据显示。
