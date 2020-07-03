## managed transaction

use promise callback chain instead of async/await

still pass transaction mannualy

## cls

all Sequelize instances, all or nothing, 可以通过稍加包装出两个独立的 Sequelize class

多事务一定是先入后出的

可以将个别查询排除出事务

用提前返回 promise 的方式提前 commit

用 throw error 的方式 rollback

无法使用 async/await 方式

