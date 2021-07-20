# 交叉熵损失函数
#### 概念
- **信息量**

    信息奠基人香农（Shannon）认为“信息是用来消除随机不确定性的东西”，也就是说衡量信息量的大小就是看这个信息消除不确定性的程度。信息量的大小与信息发生的概率成反比。概率越大，信息量越小。概率越小，信息量越大。设某一事件发生的概率P(x)，其信息量表示为：

    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB43865a50e667357d7e3c76c6e5ca2925?method=download&shareKey=cc7412b513b9082f1b5e0a914b839cf0"/>
    </div>

- **信息熵**

    信息熵也被称为熵，用来表示所有信息量的期望。

    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB505be12d9b1a508cb0de63b417f5b0f4?method=download&shareKey=6b75738730cfb3e3d9c4c371659593a9"/>
    </div>

- **KL散度**

    KL散度用来衡量两个概率分布之间的相似度。
    
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB379be4d981ba3fcce75c63b7af0b1d07?method=download&shareKey=e018141b3d97eab0a27ea090138f220c"/>
    </div>

- **交叉熵**

    KL散度拆开形似为:
    
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB120a9162659849b4353c86730d6766d8?method=download&shareKey=d2f05aefbf8ee31660ec9087811dd006"/>
    </div>
    
    由于信息熵不变，所以交叉熵越小，预测概率的分布和原始分布越接近，
    所以优化的时候按交叉熵来优化即可。

- **求导**
  
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB58f679cdda0c6d980b6061aa2d3605c5?method=download&shareKey=6def5966c6ae6ac11c80a0aa0df0c502"/>
    </div>
    
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEBecf8c8b46adf066b1ad6d7af3551eda8?method=download&shareKey=c4e86c520003b76da19706dbe2a2c753"/>
    </div>
    
- **注意点**

    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEBd1606f00dd274a37853c99cee3c42612?method=download&shareKey=02eee18c45f3ef900a3cec81a2060bfb"/>
    </div>

