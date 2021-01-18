oj前端
Linux
git clone https://github.com/QingdaoU/OnlineJudgeFE.git
cd OnlineJudgeFE
npm install

NODE_ENV=development npm run build:dll

export TARGET=https://qduoj.com/   
# target指的是后台代理服务器地址，通常为judgeserver

npm run dev

Windows
git clone https://github.com/QingdaoU/OnlineJudgeFE.git
cd OnlineJudgeFE
npm install

set NODE_ENV=development 
npm run build:dll

启动：
set TARGET=https://qduoj.com/   
set TARGET= http://192.168.3.12:8088/  

# target指的是后台代理服务器地址，通常为judgeserver

npm run dev	
