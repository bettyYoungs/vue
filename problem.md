1. 运行npm run dev时总是报错
 原因：rollup-plugin-alias对windows的兼容不好，需本地下载zip包，重新build后替换掉项目下的包文件即可解决