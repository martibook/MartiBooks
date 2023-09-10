[【2022 年】Python3 爬虫教程 - 协程的基本原理](https://cuiqingcai.com/202271.html)
 

 
**requests library** does not support async requesting 

use **aiohttp** 

`pip3 install aiohttp` 

 
&nbsp;


If encountered **error: this event loop is already running**
 
Try

`pip3 install nest-asyncio `

`import nest_asyncio `
`nest_asyncio.apply() `

&nbsp;

If failed to launch jupyter notebook by

`jupyter notebook`

Try run

`python -m notebook`