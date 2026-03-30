| `THEN`    | 串行执行 | `THEN(a, b, c)`                       |
| --------- | -------- | ------------------------------------- |
| `WHEN`    | 并行执行 | `WHEN(a, b, c)`                       |
| `IF`      | 条件判断 | `IF(condition, trueNode, falseNode)`  |
| `ELIF`    | 否则如果 | `IF(a, b).ELIF(c, d).ELSE(e)`         |
| `ELSE`    | 否则     | `IF(a, b).ELSE(c)`                    |
| `SWITCH`  | 选择分支 | `SWITCH(switchNode).to(b, c, d)`      |
| `FOR`     | 循环     | `FOR(a).DO(THEN(b, c))`               |
| `WHILE`   | 条件循环 | `WHILE(conditionNode).DO(THEN(b, c))` |
| `BREAK`   | 跳出循环 | `WHILE(a).DO(b).BREAK(c)`             |
| `CATCH`   | 异常捕获 | `CATCH(THEN(a, b)).DO(c)`             |
| `FINALLY` | 最终执行 | `THEN(a, b).FINALLY(c)`               |

